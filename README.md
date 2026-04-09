# cantonese-vocabulary-game
A fun Phaser-based game to learn Cantonese vocabular
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>廣東話詞彙小遊戲</title>
    <script src="https://cdn.jsdelivr.net/npm/phaser@3.60.0/dist/phaser-arcade-physics.min.js"></script>
    <style>
        body { margin: 0; display: flex; justify-content: center; align-items: center; height: 100vh; background-color: #f0f0f0; font-family: 'Comic Sans MS', cursive, sans-serif; }
        canvas { border-radius: 15px; box-shadow: 0 0 20px rgba(0,0,0,0.2); }
    </style>
</head>
<body>

<script>
const config = {
    type: Phaser.AUTO,
    width: 800,
    height: 600,
    backgroundColor: '#87CEEB',
    physics: { default: 'arcade' },
    scene: { preload: preload, create: create, update: update }
};

const game = new Phaser.Game(config);

let locations = [
    { key: 'school', text: '學校', x: 200, y: 150 },
    { key: 'classroom', text: '課室', x: 600, y: 150 },
    { key: 'toilet', text: '廁所', x: 200, y: 450 },
    { key: 'kitchen', text: '廚房', x: 600, y: 450 }
];

let character, currentTarget, trialCount = 0;
let feedbackText, scoreText;

function preload() {
    // Load Images - Background and location images as JPG
    this.load.image('bg', 'assets/bg.jpg');
    this.load.image('character', 'assets/student.png');
    locations.forEach(loc => {
        this.load.image(loc.key, `assets/${loc.key}.jpg`);
    });

    // Load Audio
    locations.forEach(loc => {
        this.load.audio(loc.key + '_vo', `assets/${loc.key}.mp3`);
    });
    this.load.audio('correct', 'assets/correct.mp3');
    this.load.audio('wrong', 'assets/wrong.mp3');
    this.load.audio('congrats', 'assets/congrats.mp3');
}

function create() {
    // 1. Setup Background and Locations
    this.add.image(400, 300, 'bg').setAlpha(0.3);
    
    locations.forEach(loc => {
        let zone = this.add.container(loc.x, loc.y);
        let img = this.add.image(0, 0, loc.key).setDisplaySize(200, 150).setInteractive();
        let txt = this.add.text(0, 85, loc.text, { fontSize: '32px', color: '#000', backgroundColor: '#fff' }).setOrigin(0.5);
        zone.add([img, txt]);
        zone.setSize(200, 150);
        zone.setData('key', loc.key);
    });

    // 2. Character Setup
    character = this.add.image(100, 300, 'character').setDisplaySize(100, 150).setInteractive();
    this.input.setDraggable(character);
    character.setData('startX', 100);
    character.setData('startY', 300);

    // 3. UI Elements
    scoreText = this.add.text(20, 20, '進度: 0/10', { fontSize: '24px', fill: '#000' });
    feedbackText = this.add.text(400, 300, '', { fontSize: '64px', fill: '#ff0000', stroke: '#fff', strokeThickness: 6 }).setOrigin(0.5).setVisible(false);

    // 4. Drag Logic
    this.input.on('drag', (pointer, gameObject, dragX, dragY) => {
        gameObject.x = dragX;
        gameObject.y = dragY;
    });

    this.input.on('dragend', (pointer, gameObject) => {
        checkDrop(this, gameObject);
    });

    startNewTrial(this);
}

function startNewTrial(scene) {
    if (trialCount >= 10) {
        showCongratulation(scene);
        return;
    }

    // Pick a random location
    currentTarget = Phaser.Utils.Array.GetRandom(locations);
    
    // Play Cantonese Voiceover: 「我要去 [Location]」
    scene.sound.play(currentTarget.key + '_vo');
    
    // Reset Character Position
    character.x = character.getData('startX');
    character.y = character.getData('startY');
}

function checkDrop(scene, char) {
    let matched = false;
    
    // Check if character is over the correct box
    locations.forEach(loc => {
        if (loc.key === currentTarget.key) {
            let dist = Phaser.Math.Distance.Between(char.x, char.y, loc.x, loc.y);
            if (dist < 100) {
                matched = true;
            }
        }
    });

    if (matched) {
        scene.sound.play('correct');
        trialCount++;
        scoreText.setText(`進度: ${trialCount}/10`);
        
        // Brief pause before next trial
        scene.time.delayedCall(1000, () => {
            startNewTrial(scene);
        });
    } else {
        scene.sound.play('wrong');
        // Snap back to start
        scene.tweens.add({
            targets: char,
            x: char.getData('startX'),
            y: char.getData('startY'),
            duration: 500,
            ease: 'Power2'
        });
    }
}

function showCongratulation(scene) {
    scene.sound.play('congrats');
    let overlay = scene.add.rectangle(400, 300, 800, 600, 0x000000, 0.7);
    scene.add.text(400, 300, '你好棒！\n完成練習了！', { 
        fontSize: '64px', 
        fill: '#fff', 
        align: 'center' 
    }).setOrigin(0.5);
    
    character.setVisible(false);
}

function update() {}
</script>
</body>
</html>
