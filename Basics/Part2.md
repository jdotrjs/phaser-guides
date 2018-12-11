# Phaser 3 Series: Dev 102 (part 2)

So. Your game is getting off the ground and you've noticed a few things as you
rack up some experience:

1. Game dev, and Phaser, is pretty neat and the community is super helpful;
2. There seem to be a class of mistakes that you _just keep making_ even
   though you know better;
3. Adding all those `<script>` tags to your page is starting to be a real
   annoyance and getting them in the right order is hard (or maybe impossible);
4. Javascript would be way cooler if you could use the modern extensions.

To start, thank you; we try to be helpful and like having you here. Hopefully
you'll stick around and pay that forward some day as others are learning! 😀

What about those other patterns? Well, curiously enough, they're all part of
a class of problems that plague everybody. That means two things 1) you
shouldn't feel bad about it and 2) there has bee a _lot_ of work to make life
better.

> Okay, so it's not curious at all. I started out with a collection of
  solutions I wanted to dicuss in part 2 and went through common issues that
  made sense to address which would lead into those answers, ❤️.

## Principle: Use Supportive Tools

In this article we'll talk about how the larger Javascript ecosystem has been
addressing these problems, introduce a host of new tools, and then touch on
some general programming principles that are good to know just for bonus points.

For the time being I'm not going to get into how to set up these tools as each
one is deserving of its own discussion. I will link to template projects that
use some subset of them and other relevant guides. In the beginning you don't
need to understand _how_ the tool is configured for it to be useful so bite
off a little piece at a time and slowly build up your gamedev+Javascript
muscles. When the time comes that you need to customize your setup you'll be
ready and we'll still be here to answer questions 💪.

> I'll throw the word `toolchain` around. You may not have heard it before but
  it just means "all the tools you use to build your project." Each project
  has its own toolchain that is usually driven by the personal preferences of
  the original developer.

### Code Linters

**Sugested** [ESLint][eslint]; [Setup Guide][eslint-setup]

[eslint]: TODO
[eslint-setup]: TODO

What is _lint_? It's that little bit of fabric on your shirt after doing a
load of laundry that make ... oh, you mean _code lint_.

Well, that is a bit harder and isn't as concretely defined. Most generally
it's behaviors and patterns that creep into your code that you don't want
to be there. They either make it more messy and, often, lead directly or
indirectly to errors.

Let's take a quick look at some code and find some examples. Skim through it
quickly and then we'll talk about what is going on:

```javascript
const baseAssetPath = '/assets'

function preload() {
    ['tiles/floor.png', 'sheets/pc.png', 'sheets/enemy.png'].forEach(p => {
        const filename = p.split('/')[1]
        const keyname = filename.substr(0, filenme.indexOf('.'))
        if (this.withoutNormals)
            this.load.spritesheet(keyname, `${bsaeAssetPath}/${p}`)
        else
            const path = `${baseAssetPath}/${p}`
            const normals = `${baseAssetPath}/normals/${p}`
            this.load.spritesheet(keyname, [path, normals])
    })
}

function create() {
    if (!(this.withoutNormals && this.env.dev)) {
        debugger
    }
}
```

Right, so what did you find? It loads some images and then sometimes triggers
the debugger during create. But you also should have found some mistakes:

- `baseAssetPath` has a typo when loading the spritesheet in the if block
- the if block itself doesn't have `{}` meaning the second two statements in
  the else will always execute
- when adding call to start the `debugger` in `create` we forgot that the `!`
  which means our debugger will only kick in when we do a production build

A linter is a tool that looks for patterns like this and makes sure you don't
spend time finding silly mistakes. Or at least that you spend time finding
different kinds of silly mistakes.

> Caveat: A linter will only warn that `debugger` was used, it won't magically
  be able to tell you intended for that call to happen only in dev but if you require a clean linter run before shipping your code it will keep it out of production.

### Bundlers

**Suggested**: [Webpack][webpack], [The Basics][webpack-intro], [Sample Phaser Project][webpack-phaser]

[webpack]: TODO
[webpack-intro]: TODO
[webpack-phaser]: TODO

Code Bundling as a concept is basically just taking a bunch of smaller
javascript files and putting them together in a way that a website can request
them at the same time / as a single file.

This has two major benefits one for you ard one for your player:

- As the developer you no can stop worrying about updating the HTML page
  hosting your game with a corresponding `<script>` tag or what order your
  scripts are included;
- your players will be able to load the game faster as there is significant
  overhead in making each connection to your host. For example, if you have
  30 1-2k files it's way faster to download a single 60k bundle than each
  of those component files.

### Dependency Management / 

### Code Translation (+ Polyfills!)

### Type Checking

### Editors (and Debuggers)

## A suggested toolchain

### If you're just getting started

### What I use

## Sample Projects

- Just the basics
- All the sides

# Part 2: The Bonus Section

## Principle: Rule of Three

## Principle: Do One Thing Well

## Principle: Loose coupling

## Principle: Composition over Inheritance
