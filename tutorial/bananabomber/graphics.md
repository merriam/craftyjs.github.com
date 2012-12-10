---
layout: tutorial
title: "Tiles, spritemap and animations"
---

Take a look at our sprite sheet, which you can [download here](bananabomber-sprites.png):

![bananabomber-sprites.png](bananabomber-sprites.png)

All the graphic elements have the same 16x16 pixel size because you are making a tile based game.  Some of the sprites are single pictures and some, like the player, are part of an animation. Crafty creates animations by drawing images on consecutive frames as in a movie clip or a Flash games.  A **sprite** is all the images in one animation and the whole **sprite sheet** is a RGBa PNG graphics file.   

You must cut the sprites out of the sheet and give them names for you to work with them in Crafty:

{% highlight javascript %}
//turn the sprite map into usable components
Crafty.sprite(16, "bananabomber-sprites.png", {
    grass1: [0, 0],
    grass2: [1, 0],
    grass3: [2, 0],
    grass4: [3, 0],
    flower: [0, 1],
    bush1: [0, 2],
    bush2: [1, 2],
    player: [0, 3],
    enemy: [0, 3],
    banana: [4, 0],
    empty: [4, 0]
});
{% endhighlight %}

<!--- This needs a better explanation:  the arguments do not line up to row and column. ---!>

The tile size is specified once, so we can reference the tiles by column and row. This is convenient, but if you are working with sprites of different size you can specify the exact size like this:

{% highlight javascript %}
name: [x, y, width, height]
{% endhighlight %}

Now you start writing generateWorld() one step at time.  The general idea is that you will divide the map into 16x16 pixel areas and for each decide what background and elements to put on it:

{% highlight javascript %}
//method to generate the map
function generateWorld() {
    //loop through all tiles
    for (var i = 0; i < 25; i++) {
        for (var j = 0; j < 21; j++) {

            //place grass on all tiles
            grassType = Crafty.math.randomInt(1, 4);
            Crafty.e("2D, DOM, grass" + grassType)
                .attr({ x: i * 16, y: j * 16, z:1 });
{% endhighlight %}

First you create grass background.  You create an entity for each patch of grass; use the 2D and DOM components to give the entity coordinates and the ability to draw itself to the stage; and randomly assign it one of the grass1, grass2, grass3, or grass4 components you just defined.  Any component defined via Crafty.sprite() will automatically instruct either the DOM or the Canvas component to draw it on the stage. Finally we call the attr function on the new entity that will set x and y coordinates. The **z value**, or depth, specifies what element should be in front when two or more are overlapping. The one with highest z will be on the top, so we assign the background something low.  This is similar to Flash.

You should add some bushes around the edges to provide a closed playing field for the game.  You can add this to the loop:

{% highlight javascript %}
//create a fence of bushes
if(i === 0 || i === 24 || j === 0 || j === 20)
    Crafty.e("2D, DOM, solid, bush" + Crafty.math.randomInt(1, 2))
    .attr({ x: i * 16, y: j * 16, z: 2 });
{% endhighlight %}

The drawing of the bush outside the bush edges is transparent and the bush has a higher z value, so the grass will show through under the bush.

Add some more interesting stuff on the stage! Like flowers dancing in the wind scattered over the map...
 
{% highlight javascript %}
//generate some nice flowers within the boundaries of the outer bushes
if (i > 0 && i < 24 && j > 0 && j < 20
        && Crafty.math.randomInt(0, 50) > 30
        && !(i === 1 && j >= 16)
        && !(i === 23 && j <= 4)) {
    var f = Crafty.e("2D, DOM, flower, SpriteAnimation, solid, explodable")
            .attr({ x: i * 16, y: j * 16, z: 1000 })
            .animate("wind", 0, 1, 3)
            .animate('wind', 80, -1)
            .bind('explode', function() {
                this.destroy();
            });
}
{% endhighlight %} 

Oh, what a statement!  The if expression adds flower entities to about 60% of the board inside the outside border of bushes but not in little areas of the lower left and upper right corners.  Each flower entity has 2D, DOM, and named sprite components, but with some extras.   The SpriteAnimate component adds *surprise, surprise* animation capabilities to the flower that you use by calling animate(). Your first call to animate() names an animation wind that starts in column 0, row 1 of the sprite sheet and spans 3 columns.   Your second call starts the animation that lasts 80 frames and plays forever (the -1).  You can do more advanced animations:  just look through the [SpriteAnimate documentation](http://craftyjs.com/api/SpriteAnimation.html).

The components *solid* and *explodable* are tagging components.  That is, you made up some words to mark a type of entity and there is no implentation.  Tagging works very well with Crafty's collision detection and we will have a look at that in a later article.  Similarly, binding a function to *explode* will help later.

Finally, we will finish this article by adding a grid of bushes, possibly on top of the flowers, in true bomberman style:

{% highlight javascript %}
            //grid of bushes
            if((i % 2 === 0) && (j % 2 === 0)) {
                Crafty.e("2D, DOM, solid, bush1")
                    .attr({x: i * 16, y: j * 16, z: 2000})
            }
        }
    }
}
{% endhighlight %}

Have a look at our wonderful garden:

![bananabomber-1.png](bananabomber-1.png)

<!--- Need a style marker for a JavaScript side note ---!>

Notice the % or modulo operator in JavaScript. It simply divides the left operand with the right operand and returns the remainder. Thus (i % 2 === 0) evaluates to true when i is even. 

Also notice the === instead of just ==.   The normal equality operator, ==, does type coercion before checking for equality. This means that all the following expressions return true

{% highlight javascript %}
('5' == 5)
(0 == '')
(0 == false)
(false == undefined)
("\r\n" == 0)
{% endhighlight %}

The first one is sort of okay, but the rest will come and bite you some day. That is why the father of javascript, [Douglas Crockford] (http://www.youtube.com/watch?v=hQVTIJBZook) recommends using the strict equality operator, === that will never say values of different types are equal.

In the next article we will add movement, collision detection and animation to our unit: [Keyboard input and binding to events](input-and-events)
