---
layout: post
title: "Space Invaders in JavaScript. Part 1: Modules, Canvas and Stars"
comments: true
published: true
categories: [javascript, canvas, gamedev]
---

## Prologue

In this series of posts I am going to demonstrate how one could create a sort of [Space Invaders](#TODO: link to wikipedia) game clone using pure JavaScript (well, almost pure) and browser [canvas element](#TODO:link to w3c).

There will be invaders, bullets, explosions and a lot of spacebar hits.


## The Instruments: Objects, Inheritance and Events

First thing that comes to mind when you think of creating a shooter game is the fact that the game is basically a set of objects interacting between each other.

It is easy to create an object in JavaScript, just type some curly brackets and here you go:

~~~js
var invader = {color: 'white', score_reward: 1}
~~~

Great, what about inheritance? In this project we will use a simple prototype inheritance utilities provided by the EcmaScript 5 standard.

So, how do we inherit the object properties? Here you go:

~~~js
var invader_boss = Object.create(invader)
invader_boss.color = 'red'
invader_boss.score_reward = 5
~~~

It is not cool to type ```invader_boss.something_something``` every time we want to extend the object, so let's add an utility method for that:

~~~js
Object.prototype.extend = function (properties) {
    var i;
    for (i in properties) {
        if (properties.hasOwnProperty(i)) {
            this[i] = properties[i];
        }
    }
    return this;
};
~~~

After that we can simply do this trick:

~~~js
var invader_boss = Object.create(invader).extend({
    color: 'red',
    score_reward: 5
})
~~~

Nice, isn't it?

Now, what about objects interacting between each other? A game typically includes a lot of different objects interacting in different ways, so it would be a mess to make all the objects know about the others and call their methods to perform actions. We will not do that, instead let's introduce the concept of events.

Events aka [Observer pattern](TODO: a link) is a way to decouple the code and provide a clean interface for objects to propagate changes, actions and data. For our needs it would be cool if every object could _emit_ an event and also listen to event emitted by the others.

Let's implement __emitter__, the base class for event-driven objects.

~~~js
// TODO: cleaup the code, rename stuff and get rid of crockford-style
var emitter = {
    events: [],
    on: function (name, callback, context) {
        if (!this.hasOwnProperty('events')) {
            this.events = {};
        }
        name.split(' ').forEach(function (name) {
            if (!this.events[name]) {
                this.events[name] = [];
            }
            this.events[name].push({callback: callback, context: context || null});
        }, this);
        return this;
    },
    emit: function (name, params) {
        var events = this.events[name],
            event,
            length,
            i;
        if (events) {
            event = {name: name, emitter: this, params: params};
            for (i = 0, length = events.length; i < length; i += 1) {
                if (events[i].callback.call(events[i].context, event) === false) {
                    break;
                }
            }
        }
        return this;
    },
    unsubscribe: function (name, callback, context) {
        name.split(' ').forEach(function (name) {
            if (this.events[name]) {
                this.events[name] = this.events[name].filter(function (listener) {
                    return callback && listener.callback !== callback
                        || context && listener.context !== context;
                });
            }
        }, this);
    }
}
~~~

This is exactly what we need to make the whole world move, check this out:

~~~js
var pet = {
    makeSound : function () {
        alert(this.name + ': ' + this.sound + '!')
    }
}

var dog = Object.create(animal).extend({sound: 'bark'}),
    cat = Object.create(animal).extend({sound: 'meow'})

var pets = [
    Object.create(dog).extend({name: 'Bugsy'}),
    Object.create(dog).extend({name: 'Rusty'}),
    Object.create(cat).extend({name: 'Mauser'})
]

var commander = Object.create(emitter)

commander.on('make_noise', function () {
    pets.forEach(function (pet) {
        pet.makeSound()
    })
})

commander.emit('make_noise')
~~~
