---
toc: true
comments: true
layout: post
title: Example Game
description: This is an example phaser game, look into the source code to learn something!
courses: { csa: {week: 1}}
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

    // loads all assets that will be used
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

    function create ()
    {

        let bg = this.add.image(400, 300, 'sky').setScale(5);

        platforms = this.physics.add.staticGroup();

        makePlatform(600-32/2, 30, 1920*2, platforms)

        makePlatform(220, 1000, 500, platforms)
        makePlatform(450, 600, 500, platforms)
        makePlatform(300, 50, 500, platforms)
        makePlatform(100, 500, 700, platforms)

        player = this.physics.add.sprite(100, 450, 'dude');

        player.setBounce(0.2);
        player.setCollideWorldBounds(true);
        player.setMaxVelocity(160, 400)

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

        cursors = this.input.keyboard.createCursorKeys();

        this.physics.add.collider(player, platforms);

        stars = this.physics.add.group({
            key: 'star',
            repeat: 11,
            setXY: { x: 12, y: 0, stepX: 70 }
        });

        stars.children.iterate(function (child) {
            child.setY(child.body.y + Phaser.Math.Between(0, 400))
            child.setBounceY(Phaser.Math.FloatBetween(0.4, 0.8));

        });

        this.physics.add.collider(stars, platforms);

        this.physics.add.overlap(player, stars, collectStar, null, this);

        crates = this.physics.add.group()

        this.physics.add.collider(crates, platforms, crateCheck, null, this);
        this.physics.add.collider(player, crates, crateCheck, null, this);
        this.physics.add.collider(crates, crates, crateCheck, null, this);

        scoreText = this.add.text(16, 16, 'score: 0', { fontSize: '32px', fill: '#000' });

        bombs = this.physics.add.group();

        this.physics.add.collider(bombs, crates, crateCheck, null, this);
        this.physics.add.collider(bombs, platforms);

        this.physics.add.collider(player, bombs, hitBomb, null, this);

        this.cameras.main.setBounds(0, -600*2, 800*3, 600*3);

        potionText = this.add.text(150, 340, 'Press E to drink the potion', { fontSize: '32px', fill: '#000' });

        // Try uncommenting these lines and see what happens!
        // this.physics.world.setBounds(0, -1080, 1920 * 2, 1080 * 2);
        // this.cameras.main.startFollow(player);


    }

    function update (time)
    { 
        if (cursors.left.isDown)
        {
            if (player.body.velocity.x > 0){
                    player.setAccelerationX(-300);
                }
                else{
                    player.setAccelerationX(-100);
                }
            player.anims.play('left', true);

        }
        else if (cursors.right.isDown)
        {
                if (player.body.velocity.x < 0){
                    player.setAccelerationX(300);
                }
                else{
                    player.setAccelerationX(100);
                }
            player.anims.play('right', true);
        }
        else if (cursors.down.isDown && !player.body.touching.down){
            player.setAccelerationY(600)
        }
        else
        {
            player.setAccelerationY(0);
            if ((player.body.velocity.x > 0.1 || player.body.velocity.x < -0.1) && player.body.touching.down ){
                player.setAccelerationX(-400*player.body.velocity.x/100);
            }
            else {
                player.setAccelerationX(0);
            }
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

        if (cursors.up.isDown && player.body.touching.down){
            player.setVelocityY(-400);
        }

        if(cursors.shift.isDown){
            var crate = crates.create(Phaser.Math.Between(0, 800), Phaser.Math.Between(0, 300), 'crate')
            crate.setScale(.2);
            crate.setBounce(.4)
            crate.setCollideWorldBounds(true);
            crateList.push(crate)
        }
        if (potionThere){
            showText = false;
            potions.forEach((c) => {
                c.movement(time)
                if (checkOverlap(c, player)){
                    nearPotion(c, player)
                    potionText.setVisible(true);
                    showText = true;
                }
        })
        }
        potionText.setVisible(showText);

        crateList.forEach((c) => {
            if (c.body.touching.down){
                c.setDrag(30)
            }else{
                c.setDrag(0)
            }
        })
    }

    function collectStar (player, star)
    {
        star.disableBody(true, true);

        score += 10;
        scoreText.setText('Score: ' + score);

        if (stars.countActive(true) === 0)
        {
            stars.children.iterate(function (child) {

                child.enableBody(true, child.x, Phaser.Math.Between(0, 400), true, true);

            });

            var x = (player.x < 400) ? Phaser.Math.Between(400, 800) : Phaser.Math.Between(0, 400);

            var bomb = bombs.create(x, 16, 'bomb');
            bomb.setBounce(1);
            bomb.setCollideWorldBounds(true);
            bomb.setVelocity(Phaser.Math.Between(-200, 200), 20);

            if (score % 240 == 0){
                var crate = crates.create(Phaser.Math.Between(0, 800), Phaser.Math.Between(0, 300), 'crate')
                crate.setScale(.2);
                crate.setBounce(.4)
                crate.setCollideWorldBounds(true);
                crateList.push(crate)
            }

        }

    }

    function hitBomb (player, bomb)
    {
        this.physics.pause();

        player.setTint(0xff0000);

        player.anims.play('turn');

        gameOver = true;
    }
    function crateCheck(hitter, crate){
        if (hitter.body.velocity.y < 150){
            hitter.setVelocityY(0);
            hitter.setAccelerationY(0);
        }
        else {
            var x = crate.body.position.x + 16.875
            var y = crate.body.position.y + 16.875
            crate.destroy()
            crateList.splice(crateList.indexOf(crate), 1); 
            cratePart = this.physics.add.group({
                key: 'cratePart',
                repeat: Phaser.Math.Between(2,3),
                setXY: {x: x, y: y},
            })
            cratePart.children.iterate(function (child) {
                child.setScale(.3);
                child.setVelocity(Phaser.Math.Between(-75,75), -150)
                child.setAngularVelocity(Phaser.Math.Between(-300,300))
            })
            var randNum = Math.floor(Math.random()*4)
            potion = new Potion(this, x, y + 4.5, randNum);
            potion.setScale(0.075)
            this.add.existing(potion);
            potions.push(potion)
            potionThere = true;
        }
    }

    function makePlatform(y, x, width, group){
        var groundWidth = 400;
        var groundHeight = 32;
        if (groundWidth >= 400){
            var x1 = x;
            var x2 = x + width;
            var sw = true;
            while (x1 < x2){
                if (sw){
                    group.create(x1, y, "ground");
                    x1 = x1 + groundWidth;
                }
                else {
                    group.create(x2-groundWidth, y, "ground");
                    x2 = x2 - groundWidth;
                }
                sw = !sw;
            }
        }
    }

    function checkOverlap(spriteA, spriteB) {
	    var boundsA = spriteA.getBounds();
	    var boundsB = spriteB.getBounds();
	    return Phaser.Geom.Intersects.RectangleToRectangle(boundsA, boundsB);
	}

    function nearPotion(potion, player){
        if(e){
            console.log(player.scaleY)
            potions.splice(potions.indexOf(potion), 1); 
            potion.destroy();
            potion.drinkAction(player);
        }
    }

    class Potion extends Phaser.Physics.Arcade.Sprite{
        constructor(scene, x, y, num){
            var asset;
            var type;
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
            super(scene, x, y, asset)
            this.type = type;
            this.movementNum = Math.floor(Math.random() * 2)
            this.movementMod = Math.random() * 2
        }
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
        drinkAction(player){
            switch (this.type){
                case 'red':
                    player.scaleY += 0.15
                    player.scaleX += 0.15
                    if (player.body.touching.down){
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
