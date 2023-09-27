---
toc: true
comments: true
layout: post
title: Example Game
description: This is an example phaser game, look into the source code to learn something!
courses: { ToC: {week: 1}}
type: tangibles
---

<script src="//cdn.jsdelivr.net/npm/phaser@3.11.0/dist/phaser.js"></script>
<style type="text/css">
    body {
        margin: 0;
    }
    canvas {
        padding: 0;
        margin: auto;
        display: block;
        width: 800px;
        height: 600px;
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
    }

</style>

<script type="text/javascript">
    // configuration for the phaser game. Try changing width, height, gravity, etc, to see what happends and look into the documention for more info. 
    var config = {
        type: Phaser.AUTO,
        width: 800,
        height: 600,
        physics: {
            default: 'arcade',
            arcade: {
                gravity: { y: 300 },
                debug: false
            }
        },
        scene: { // This part is important, it defines the 3 necesarry functions for a phaser game to run. 
            preload: preload,
            create: create,
            update: update
        }
    };

    // initializes the phaser game and passes through the configuation
    var game = new Phaser.Game(config);

    // preload function, necessary for phaser. Loads any assets
    function preload ()
    {
        this.load.setPath('{{site.baseurl}}/assets/images/')
        this.load.image('sky', 'sky.png');
        this.load.image('ground', 'platform.png');
        this.load.image('star', 'star.png');
        this.load.image('bomb', 'bomb.png');
        this.load.spritesheet('dude', 
            '/dude.png',
            { frameWidth: 32, frameHeight: 48 }
        );
        this.load.image('crate', 'crate.png')
        this.load.image('cratePart', 'cratePart.png')
        this.load.image('rPotion', 'pt1.png')
        this.load.image('bPotion', 'pt2.png')
        this.load.image('gPotion', 'pt3.png')
        this.load.image('yPotion', 'pt4.png')
    }

    // initializes global variabls
    var potionThere = false;
    var player;
    var stars;
    var platforms;
    var cursors;
    var score = 0;
    var scoreText;
    var potions = [];
    var crateList = [];
    var showText;
    var e = false;

    // create function, necessary for phaser. Loads any assets
    function create ()
    {
        // Adds the sky image to the background. "this" is the phaser game object. .setScale(5) is a method which scales up the sky by 5.
        let bg = this.add.image(400, 300, 'sky').setScale(5);

        // Creates the physics group for the platforms. A group allows all children of the group to have the same characteristics. Static means the group is immovable.
        platforms = this.physics.add.staticGroup();

        // A function I created to make dynamicly sized platforms based on the single picture I have without using the scaling feature
        makePlatform(600-32/2, 30, 1920*2, platforms)
        makePlatform(220, 1000, 500, platforms)
        makePlatform(450, 600, 500, platforms)
        makePlatform(300, 50, 500, platforms)
        makePlatform(100, 500, 700, platforms)


        // Initializes the main player and the dude sprite
        player = this.physics.add.sprite(100, 450, 'dude');

        // Defines player settings like bounce, colliding to world bounds, and the maximum velocity
        player.setBounce(0.2);
        player.setCollideWorldBounds(true);
        player.setMaxVelocity(160, 400)

        // Creates animations based on the dude spritesheet. These animations are properties of phaser, not the sprite itself, so it can be used by any sprite with the same model
        this.anims.create({
            key: 'left',
            frames: this.anims.generateFrameNumbers('dude', { start: 0, end: 3 }),
            frameRate: 10,
            repeat: -1
        });

        this.anims.create({
            key: 'turn',
            frames: [ { key: 'dude', frame: 4 } ],
            frameRate: 20
        });

        this.anims.create({
            key: 'right',
            frames: this.anims.generateFrameNumbers('dude', { start: 5, end: 8 }),
            frameRate: 10,
            repeat: -1
        });

        // Initializes cursors, which allows you to check the player keyboard
        cursors = this.input.keyboard.createCursorKeys();

        // Creates a collider between the player and platforms, meaning they will stop each other if they hit each other
        this.physics.add.collider(player, platforms);

        // Initializes stars group. Repeats 12 times to create individual stars. They have the same y value, but stepX increases the x value by 70 each time
        stars = this.physics.add.group({
            key: 'star',
            repeat: 11,
            setXY: { x: 12, y: 0, stepX: 70 }
        });
        
        // iterates through the children of the stars group (the individual stars) to set their y value randomly and their bounce value
        stars.children.iterate(function (child) {
            child.setY(child.body.y + Phaser.Math.Between(0, 400))
            child.setBounceY(Phaser.Math.FloatBetween(0.4, 0.8));

        });

        // adds a collider between the stars and platforms
        this.physics.add.collider(stars, platforms);

        // adds and overlap between the stars and player. collectStar is the method that will be run when the player and stars overlap. Even though collectStar does not have parenthesis, it is a method, we're just passing through the method rather than running it
        this.physics.add.overlap(player, stars, collectStar, null, this);

        // Initializes the crates group
        crates = this.physics.add.group()

        // Adds all of the necessary colliders for the crates. Runs the crateCheck method on the collision
        this.physics.add.collider(crates, platforms, crateCheck, null, this);
        this.physics.add.collider(player, crates, crateCheck, null, this);
        this.physics.add.collider(crates, crates, crateCheck, null, this);

        // initializes the text to display score
        scoreText = this.add.text(16, 16, 'score: 0', { fontSize: '32px', fill: '#000' });

        // initializes the bombs goups
        bombs = this.physics.add.group();

        // Adds necessary colliders for bombs, runs crate check on collision with a crate and hit bomb on collision with a bomb
        this.physics.add.collider(bombs, crates, crateCheck, null, this);
        this.physics.add.collider(bombs, platforms);
        this.physics.add.collider(player, bombs, hitBomb, null, this);

        // sets the bounds of the camera
        this.cameras.main.setBounds(0, -600*2, 800*3, 600*3);

        // initializes text to display how to use a potion
        potionText = this.add.text(150, 340, 'Press E to drink the potion', { fontSize: '32px', fill: '#000' });

        // Try uncommenting these lines and see what happens!
        // this.physics.world.setBounds(0, -1080, 1920 * 2, 1080 * 2);
        // this.cameras.main.startFollow(player);
    } // end of create function

    // update function, necessary for phaser. Actually does stuff
    function update (time)
    { 
        // sets player's acceleration towards the left if the left arrow is pressed
        if (cursors.left.isDown)
        {
            if (player.body.velocity.x > 0){ // checks which direction the player is moving and changes the acceleration based on that, this creates a friction like effect
                    player.setAccelerationX(-300);
                }
                else{
                    player.setAccelerationX(-100);
                }
            player.anims.play('left', true); // plays the animation called 'left'

        }
        // sets player's acceleration towards the left if the left arrow is pressed
        else if (cursors.right.isDown)
        {
                if (player.body.velocity.x < 0){ 
                    player.setAccelerationX(300); // checks which direction the player is moving and changes the acceleration based on that, this creates a friction like effect
                }
                else{
                    player.setAccelerationX(100);
                }
            player.anims.play('right', true); // plays the animation called 'right'
        }
        // accelerates the player down if they're pressing down and the bottom of the sprite isn't touching anything
        else if (cursors.down.isDown && !player.body.touching.down){
            player.setAccelerationY(600)
        }

        else
        {
            player.setAccelerationY(0); // sets the acceleration to 0, does not stop gravity from acting on the player
            if ((player.body.velocity.x > 0.1 || player.body.velocity.x < -0.1) && player.body.touching.down ){ // checks if the player was moving in a direction and is on the ground, sets acceleratoin based on their velocity, creates friction effect
                player.setAccelerationX(-400*player.body.velocity.x/100);
            } // sets acceleration on the x to 0 if the player isn't moving fast enough
            else {
                player.setAccelerationX(0);
            } // plays animations based on which direction the player is moving
            if (player.body.velocity.x > 7){
                player.anims.play('right', !player.body.touching.down)
            }
            else if (player.body.velocity.x < -7){ 
                player.anims.play('left', !player.body.touching.down)
            }
            else{
                player.anims.play('turn');
            }
        }
        
        // jumps if the user presses up and the bottom of the player sprite is touching something 
        if (cursors.up.isDown && player.body.touching.down){
            player.setVelocityY(-400);
        }

        // function just to test the crates, just makes a bunch of crates when shift is being pressed
        // if(cursors.shift.isDown){
        //     var crate = crates.create(Phaser.Math.Between(0, 800), Phaser.Math.Between(0, 300), 'crate')
        //     crate.setScale(.2);
        //     crate.setBounce(.4)
        //     crate.setCollideWorldBounds(true);
        //     crateList.push(crate)
        // }

        // Checks the potionThere variable which is set to true if there are potions in the game
        if (potionThere){
            showText = false;
            // iterates through the list of potions 
            potions.forEach((c) => {
                c.movement(time) // runs the potion's movement command
                if (checkOverlap(c, player)){ // checks if the potion is overlapping with the player using a different overlap function 
                    nearPotion(c, player)
                    potionText.setVisible(true);
                    showText = true; // variable to later set the potion text to
                }
            })
        } 
        // sets the potion text to whatever the showText variable is
        potionText.setVisible(showText); 
        // iterates through the list of crates
        crateList.forEach((c) => {
            if (c.body.touching.down){ // checks if the crate is touching the ground, if it is sets the drag
                c.setDrag(30) // In this case, drag is used to create friction for the crates. This effect could have been used for the player as well
            }else{
                c.setDrag(0)
            }
        })
    } // end of update function

    // function ran when a player overlaps with a star. Parameters are player and star, these are always passed through when a function is run through a collide or overlap method
    function collectStar (player, star)
    {
        star.disableBody(true, true); // removes the star
        score += 10;
        scoreText.setText('Score: ' + score);
        if (stars.countActive(true) === 0) // checks if there are any stars. Is there anywhere else in the code this could have been used 
        {   
            // turns all of the stars back on
            stars.children.iterate(function (child) {
                child.enableBody(true, child.x, Phaser.Math.Between(0, 400), true, true);

            });
            // generates a random x value thats on the opposite side of the map as the player
            var x = (player.x < 400) ? Phaser.Math.Between(400, 800) : Phaser.Math.Between(0, 400);
            // creates a new bomb and sets its variables
            var bomb = bombs.create(x, 16, 'bomb');
            bomb.setBounce(1);
            bomb.setCollideWorldBounds(true);
            bomb.setVelocity(Phaser.Math.Between(-200, 200), 20);

            if (score % 240 == 0){ // checks if it's been 2 rounds 
                // creates a new crate at a random x and y value
                var crate = crates.create(Phaser.Math.Between(0, 800), Phaser.Math.Between(0, 300), 'crate')
                crate.setScale(.2);
                crate.setBounce(.4)
                crate.setCollideWorldBounds(true);
                crateList.push(crate)
            }
        }

    }

    // function that stops the game when a player is hit by a bomb
    function hitBomb (player, bomb)
    {
        this.physics.pause();
        player.setTint(0xff0000);
        player.anims.play('turn');
        gameOver = true;
    }
    
    // function run if crate is collided with
    function crateCheck(hitter, crate){
        // checks if the y velocity of whatever hit the crate is below a certain value
        if (hitter.body.velocity.y < 150){
            hitter.setVelocityY(0); // sets the velocity and acceleration of whatever hit the crate to 0. Fixes a big where the player went through the crate
            hitter.setAccelerationY(0);
        }
        // runs if the hitter is fast enough
        else {
            // new x and y values for where the player hit, since the potion should spawn in the middle of the crate and the crates position is based in the corner
            var x = crate.body.position.x + 16.875
            var y = crate.body.position.y + 16.875
            // gets rid of the crate and removes it from the cratelist
            crate.destroy()
            crateList.splice(crateList.indexOf(crate), 1); 
            // creates crateParts
            cratePart = this.physics.add.group({
                key: 'cratePart',
                repeat: Phaser.Math.Between(2,3),
                setXY: {x: x, y: y},
            })
            // iterates through cratePart children to give them random velocity and angular velocity
            cratePart.children.iterate(function (child) {
                child.setScale(.3);
                child.setVelocity(Phaser.Math.Between(-75,75), -150)
                child.setAngularVelocity(Phaser.Math.Between(-300,300))
            })
            // creates a random number for the potion to use
            var randNum = Math.floor(Math.random()*4)
            // creates new potion using object definition syntax
            potion = new Potion(this, x, y + 4.5, randNum);
            potion.setScale(0.075)
            // adds potion to the game, necessary when creating an object through our own class
            this.add.existing(potion);
            // adds the potion to the potions list
            potions.push(potion)
            potionThere = true;
        }
    }

    // function to create platforms. Essentially, it sets values for the two x values that the platform will be between, then adds a platform to each end until the two ends reach each other
    function makePlatform(y, x, width, group){
        var groundWidth = 400; // length of ground image
        var groundHeight = 32; // height of ground image
        if (width >= 400){ // checks if width is greater than the ground width
            var x1 = x;
            var x2 = x + width;
            var sw = true;
            while (x1 < x2){
                if (sw){
                    group.create(x1, y, "ground"); // creates a new element in the group that was passed through
                    x1 = x1 + groundWidth;
                }
                else {
                    group.create(x2-groundWidth, y, "ground");  // creates a new element in the group that was passed through
                    x2 = x2 - groundWidth;
                }
                sw = !sw;
            }
        }
        else { // runs if the platform width is less than the size of the picture (bugged)
            var scale = width/groundWidth;
            var newHeight = 32 * scale
            var y1 = y;
            var y2 = y + 32;
            var sw = true;
            while (y1 < y2){
                if (sw){
                    group.create(x, y1, "ground").setScale(scale).setSize(width, newHeight); // creates a new element in the group that was passed through. Changes the scale of the sprite and the size. However the size doesn't work as intended, the hitbox of the model stays outside of the image. 
                    y1 = y1 + newHeight;
                }
                else {
                    group.create(x, y2 - newHeight, "ground").setScale(scale).setSize(width, newHeight);  // creates a new element in the group that was passed through. Changes the scale of the sprite and the size.
                    y2 = y2 - newHeight;
                }
                sw = !sw;
            }
        }
    }

    // function to check for overlapping sprites. Used when unable to use the overlap detector built in because the detection needs to be continous or more dynamic
    function checkOverlap(spriteA, spriteB) {
	    var boundsA = spriteA.getBounds();
	    var boundsB = spriteB.getBounds();
	    return Phaser.Geom.Intersects.RectangleToRectangle(boundsA, boundsB);
	}

    // function run when a player is near a potion. if e has been pressed (checked by a boolean)
    function nearPotion(potion, player){
        if(e){
            potions.splice(potions.indexOf(potion), 1); 
            potion.destroy();
            potion.drinkAction(player);
        }
    }
    
    // Potion class definition
    class Potion extends Phaser.Physics.Arcade.Sprite{
        // constructor, contains all of the basic parameters of a sprite and our own custom one, num
        constructor(scene, x, y, num){
            // variable definitions
            var asset;
            var type;
            
            // Sets the asset and potion type randomly based on the number passed through
            switch (num){
                case 0:
                    asset = 'rPotion'
                    type = 'red'
                    break;
                case 1:
                    asset = 'bPotion'
                    type = 'blue'
                    break;
                case 2:
                    asset = 'gPotion'
                    type = 'green'
                    break;
                case 3:
                    asset = 'yPotion'
                    type = 'yellow'
                    break;
            }
            // calls the "super" constructor, the constructor of the parent class
            super(scene, x, y, asset)
            // sets variables necessary for the potion to function
            this.type = type;
            this.movementNum = Math.floor(Math.random() * 2)
            this.movementMod = Math.random() * 2
        }
        // Method to move the potion, sets the position based on a sin or cosine function and the time. Includes elements of randomness to make the potions move differently
        movement(time){
            switch(this.movementNum){
                case 0:
                    this.y = this.y + 0.2 * Math.sin(time/400 * this.movementMod);
                    break;
                case 1:
                    this.y = this.y + 0.2 * Math.cos(time/400 * this.movementMod);
                    break;
            }
        }
        
        // Method to perform the action whenever the player drinks the potion. Passes through the player who drank the potion
        drinkAction(player){
            // potion has different effects based on the type. Try to add an effect yourself!
            switch (this.type){
                case 'red':
                    player.scaleY += 0.15
                    player.scaleX += 0.15
                    if (player.body.touching.down){ // moves player upwards so they aren't inside of the ground when they drink the potion. Also ends up kind of looking intentional like a jump
                        player.body.setVelocityY(-100);
                    }
                    break;
                case 'blue':
                    
                    break;
                case 'yellow':
                    
                    break;
                case 'green':
                    
                    break;
            }
        }

    }

    // event listeners for checking whether e is pressed or not
    document.addEventListener("keydown", function(event){
        if (event.keyCode == 69){
            e = true;
        }
    })
    document.addEventListener("keyup", function(event){
        if (event.keyCode == 69){
            e = false;
        }
    })
</script>
