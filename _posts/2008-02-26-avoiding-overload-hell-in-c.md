---
layout: post
title: "Avoiding Overload Hell in C#"
categories: c-sharp code
---

C# [lacks default parameters](http://blogs.msdn.com/csharpfaq/archive/2004/03/07/85556.aspx). The C# answer to default params is
overloaded methods. With a lot of options, this can quickly [scale to
hell](http://msdn2.microsoft.com/en-us/library/system.drawing.graphics.drawimage.aspx). Even if it had default parameters, they are pretty limited too.
This post talks a bit about a couple of other solutions I found to the problem
that avoid the combinatorial sea of overloads.

## The Problem

For kicks, I wrote a little [curses](http://en.wikipedia.org/wiki/Curses_(programming_library))-like terminal library in C#. It only
really has a few core methods: `Write()` for writing text, `Fill()` to fill in
a space, and `Draw()` to draw lines and boxes using awesome [ASCII lines](http://academic.evergreen.edu/projects/biophysics/technotes/program/ascii_ext-pc.htm).

The tricky part is that each of those can take a bunch of different
parameters. For our talk, we'll simplify it down:

  1. *Position:* You can specify where on the screen to write using a `Point`, x and y coordinates, or nothing to write at the current cursor position.

  2. *Color:* You can specify both the fore- and background color, just the foreground color, or neither to use the current colors.

And that's it. Three methods, and two kinds of parameters with three options
each.

## Overloads: The Vanilla Solution

So the normal way to address this is by overloading `Write()`, `Fill()`, and
`Draw()`. If we go with the vanilla overload solution, we have 27 overloaded
methods to implement every combination. Using the code looks like this:

```csharp
Terminal terminal = new Terminal();

terminal.Write("foo");
terminal.Fill(1, 2, Color.Red);
terminal.Draw(Color.Green, Color.Blue);
```

Not bad. But implementing it looks like:

```csharp
public class Terminal
{
    public void Write(Point pos,
        Color fore, Color back, string text) { /* stuff... */ }
    public void Write(int x, int y,
        Color fore, Color back, string text) { /* stuff... */ }
    public void Write(Color fore,
        Color back, string text) { /* stuff... */ }

    public void Write(Point pos,
        Color fore, string text) { /* stuff... */ }
    public void Write(int x, int y,
        Color fore, string text) { /* stuff... */ }
    public void Write(Color fore,
        string text) { /* stuff... */ }

    public void Write(Point pos,
        string text) { /* stuff... */ }
    public void Write(int x, int y,
        string text) { /* stuff... */ }
    public void Write(string text) { /* stuff... */ }

    public void Fill(Point pos,
        Color fore, Color back) { /* stuff... */ }
    public void Fill(int x, int y,
        Color fore, Color back) { /* stuff... */ }
    public void Fill(Color fore,
        Color back) { /* stuff... */ }

    public void Fill(Point pos,
        Color fore) { /* stuff... */ }
    public void Fill(int x, int y,
        Color fore) { /* stuff... */ }
    public void Fill(Color fore) { /* stuff... */ }

    public void Fill(Point pos) { /* stuff... */ }
    public void Fill(int x, int y) { /* stuff... */ }
    public void Fill() { /* stuff... */ }

    public void Draw(Point pos,
        Color fore, Color back) { /* stuff... */ }
    public void Draw(int x, int y,
        Color fore, Color back) { /* stuff... */ }
    public void Draw(Color fore,
        Color back) { /* stuff... */ }

    public void Draw(Point pos,
        Color fore) { /* stuff... */ }
    public void Draw(int x, int y,
        Color fore) { /* stuff... */ }
    public void Draw(Color fore) { /* stuff... */ }

    public void Draw(Point pos) { /* stuff... */ }
    public void Draw(int x, int y) { /* stuff... */ }
    public void Draw() { /* stuff... */ }
}
```

Lame! Worse, if you start adding more options, it goes up combinatorially. 27
is just for our *toy* terminal. For my actual terminal lib, it would take
*324* overloaded methods to support every combination. There has to be a
better way.

## Variable Argument Lists: Who Needs Strong Typing Anyway?

It's true that C# doesn't support default parameters, but it *does* support
variable length argument lists. Maybe that's a solution that lets us pass in a
variety of options without having to overload. Calling it would still look
like:

```csharp
Terminal terminal = new Terminal();

terminal.Write("foo");
terminal.Fill(1, 2, Color.Red);
terminal.Draw(Color.Green, Color.Blue);
```

But the class itself just looks like:

```csharp
public class Terminal
{
    public void Write(string text,
        params object[] options) { /* stuff... */ }
    public void Fill(params object[] options) { /* stuff... */ }
    public void Draw(params object[] options) { /* stuff... */ }
}
```

That doesn't look so bad. Except when you try to implement one of the methods:

```csharp
public void Write(string text, params object[] options)
{
    Point pos = mCurrentPos;
    Color? foreColor = null;
    Color? backColor = null;
    int? x = null;
    int? y = null;

    foreach (object obj in options)
    {
        if (obj is Point)
        {
            pos = (Point)obj;
        }
        else if (obj is Color)
        {
            if (foreColor == null) foreColor = (Color)obj;
            else backColor = (Color)obj;
        }
        else if (obj is int)
        {
            if (x == null) x = (int)obj;
            else y = (int)obj;
        }
    }

    if (x == null) x = mCurrentX;
    if (y == null) y = mCurrentY;

    // write using pos, color, etc.
}
```

That's… just… no. And that doesn't even have any error handling. The following
is totally valid:

```csharp
terminal.Write("foo", 17, Color.White, Color.Aqua,
    Color.Beige, "wtf?");
```

OK, scratch that.

## Parameter Object: Strong Typing is Strong

So we know we want strong typing. How about if we bundle all of the parameters
into a parameter object? This is basically the [`ProcessStartInfo`](http://msdn2.microsoft.com/en-us/library/system.diagnostics.processstartinfo_properties.aspx)
solution.

We define a class something like:

```csharp
public class TerminalParams
{
    // use nullable so that null = default value
    public Point? Pos;
    public Color? Fore;
    public Color? Back;
}
```

And our `Terminal` looks something like:

```csharp
public class Terminal
{
    public void Write(TerminalParams paramObj, string text)
        { /* stuff... */ }
    public void Fill(TerminalParams paramObj)
        { /* stuff... */ }
    public void Draw(TerminalParams paramObj)
        { /* stuff... */ }
}
```

In C# 3.0 with [object initializers](http://weblogs.asp.net/scottgu/archive/2007/03/08/new-c-orcas-language-features-automatic-properties-object-initializers-and-collection-initializers.aspx), we can call it something like:

```csharp
Terminal terminal = new Terminal();

terminal.Write(new TerminalParams(), "foo");
terminal.Fill(new TerminalParams
        { Pos = new Point(1, 2), Fore = Color.Red });
terminal.Draw(new TerminalParams
        { Fore = Color.Green, Back = Color.Blue });
```

I won't say that's the greatest thing ever, but it's not too bad. Of course,
if you're not on 3.0 yet, you're up a creek. All you've basically accomplished
is moved your overloads to constructor overloads in `TerminalParams`. And I
still think there's a little too much… junk… in those lines of code. "`new
TerminalParams {`" and "`new Point(...)`", none of that really adds value.

## Parameter Groups

Really, the overloaded methods do have the best _calling convention_ so far.
Let's look at it a little closer:

```csharp
terminal.Write(1, 2, Color.Blue, Color.Red, "foo");
```

While that's an uninterrupted list of parameters, they're conceptually grouped
like this:

```csharp
terminal.Write(  1, 2,     Color.Blue, Color.Red, "foo"  );
//              (position) (color              )  (text)
```

How cool would it be if we could actually group the parameters like that in
the code? Something like:

```csharp
terminal.Write(1, 2)(Color.Blue, Color.Red)("foo");
```

C# doesn't have functors, but maybe we're on to something.

*Insert several hours of mucking around with properties that return delegates
that return delegates, etc.*

Or… not. But let's not give up yet. Let's try re-arranging things:

```csharp
terminal.Write("foo")(1, 2)(Color.Blue, Color.Red);
```

This is a little better because it puts the method (`Write()`) next to the one
parameter it really needs, the string. The problem is that that line, if it
were possible to write, would execute from left to right. So we would have to
write before we looked at the parameters.

Let's do some more shuffling:

```csharp
terminal(1, 2)(Color.Blue, Color.Red).Write("foo");
```

This looks promising, but we're stuck with the fact that C# doesn't have
functors. We can't do `terminal()`. Is there anything that sort of *looks*
like `()`?

## You're Using What?!

So the question is, "what kind of grouping construct can we define at the
object level?" Astute readers are already thinking it: *indexers*. Right now,
some of you may be shuddering in horror at the weirdness we're about to
unleash, but let's do it anyway. Can we make all of these lines of code work:

```csharp
terminal.Write("foo");
terminal[new Point(3, 4)].Write("foo");
terminal[1, 2][Color.Red].Write("foo");
terminal[Color.Brown].Write("foo");
terminal[Color.Green, Color.Blue].Draw();
```

And can we make them work all at the same time? Not only that, can we make the
following *not* work:

```csharp
// bad, cannot specify position twice
terminal[1, 2][new Point(3, 4)].Write("foo");

// bad, cannot specify half a position
terminal[1].Write("foo");

// sorta bad, for consistency would always like
// to have position before color
terminal[Color.Brown][1, 2].Write("foo");
```

It turns out, we can. The core idea is to define a chain of interfaces that
inherit from each other.

```csharp
public interface ITerminalMethods
{
    void Write(string text);
    void Fill();
    void Draw();
}

public interface ITerminalColor : ITerminalMethods
{
    ITerminalMethods this[Color fore] { get; }
    ITerminalMethods this[Color fore, Color back] { get; }
}

public interface ITerminalPosColor : ITerminalColor
{
    ITerminalColor this[Point pos] { get; }
    ITerminalColor this[int x, int y] { get; }
}

public interface ITerminal : ITerminalPosColor { }
```

Each one represents one of the parameter types: position, color, etc. The
innermost one, `ITerminalMethods`, contains the actual methods, `Write()`,
`Fill()`, etc.

The other interfaces define indexers for each parameter type that return the
next interface up the chain. The position interface, `ITerminalPosColor`, has
indexers that return the color interface, `ITerminalColor`, which has indexers
which return the method interface, `ITerminalMethods`. Since the interfaces
inherit from each other too, you can also skip a step. You can go from
`ITerminalPosColor` straight to `ITerminalMethods` because `ITerminalColor`
interface inherits from it.

Calling it looks like:

```csharp
terminal.Write("foo");
terminal[new Point(3, 4)].Write("foo");
terminal[1, 2][Color.Red].Write("foo");
terminal[Color.Brown].Write("foo");
terminal[Color.Green, Color.Blue].Draw();
```

This is all well and good for *defining* it, but is it possible to actually
*implement* this thing?

We can do that too. The idea behind that is that the Terminal class really
contains two kinds of state: the core "set of characters on screen" state and
the "temporary state" that can be overridden by these indexers: current color
and position. Then we let a Terminal create a shallow clone of itself that
*shares* the core state, but has a *copy* of the temporary state that can be
changed. All of the indexers then basically create new Terminals that write to
the same core terminal data but keep their own copy of the position and color.
Like so:

```csharp
public class TerminalData
{
    // terminal core data:
    public char[] Characters;
    public Color[] ForeColors;
    public Color[] BackColors;
}

public class Terminal : ITerminal
{
    public Terminal() { mData = new TerminalData(); }

    #region ITerminalPosColor Members

    public ITerminalColor this[Point pos]
    {
        get { return new Terminal(mData, pos, mFore, mBack); }
    }

    public ITerminalColor this[int x, int y]
    {
        get { return new Terminal(mData,
            new Point(x, y), mFore, mBack); }
    }

    #endregion

    #region ITerminalColor Members

    public ITerminalMethods this[Color fore]
    {
        get { return new Terminal(mData, mPos, fore, mBack); }
    }

    public ITerminalMethods this[Color fore, Color back]
    {
        get { return new Terminal(mData, mPos, fore, back); }
    }

    #endregion

    #region ITerminalMethods Members

    public void Write(string text) { /* stuff... */ }
    public void Fill() { /* stuff... */ }
    public void Draw() { /* stuff... */ }

    #endregion

    private Terminal(TerminalData data, Point pos,
        Color fore, Color back)
    {
        mData = data;
        mPos  = pos;
        mFore = fore;
        mBack = back;
    }

    private TerminalData mData;
    private Point mPos;
    private Color mFore;
    private Color mBack;
}
```

## Is it Worth Doing?

So this is how my little terminal library works, and I've grown pretty fond of
it. However, this is a personal project, so my willingness to deal with very
idiomatically strange code is pretty high. (Ask me about [`NotNull<T>`](http://journal.stuffwithstuff.com/2008/04/08/whats-the-opposite-of-nullable/)
sometime.)

In a production environment, this might be too strange looking for some teams.
But maybe if it gets popularized enough what was once strange will become
familiar.
