# LibGDX Space Clicker Game - Complete Implementation Guide

## 🚀 Project Overview
A space-themed PVE clicker game where players click to attack enemies, earn money, and upgrade their spaceship with enhanced damage, health, and speed.

## 🛠️ Project Setup

### 1. Create LibGDX Project
```bash
# Download LibGDX Project Generator from: https://libgdx.com/dev/setup/
# Or use gdx-liftoff: https://github.com/tommyettinger/gdx-liftoff

# Project Settings:
# Name: SpaceClicker
# Package: com.spaceclicker.game
# Game Class: SpaceClickerGame
# Platforms: Desktop
# Extensions: Box2D, Freetype
```

### 2. Project Structure
```
SpaceClicker/
├── core/
│   └── src/com/spaceclicker/game/
│       ├── SpaceClickerGame.java
│       ├── GameScreen.java
│       ├── MenuScreen.java
│       ├── entities/
│       │   ├── Player.java
│       │   ├── Enemy.java
│       │   └── Projectile.java
│       ├── systems/
│       │   ├── UpgradeSystem.java
│       │   ├── EnemySpawner.java
│       │   └── ParticleManager.java
│       ├── ui/
│       │   ├── GameUI.java
│       │   └── UpgradeUI.java
│       └── utils/
│           ├── Assets.java
│           └── GameData.java
├── desktop/
└── assets/
    ├── images/
    ├── sounds/
    └── particles/
```

## 🎮 Core Implementation

### 1. Main Game Class (SpaceClickerGame.java)
```java
package com.spaceclicker.game;

import com.badlogic.gdx.Game;
import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.graphics.GL20;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.spaceclicker.game.utils.Assets;

public class SpaceClickerGame extends Game {
    public SpriteBatch batch;
    public Assets assets;
    
    @Override
    public void create() {
        batch = new SpriteBatch();
        assets = new Assets();
        assets.load();
        
        // Wait for assets to load
        assets.manager.finishLoading();
        
        // Start with GameScreen
        setScreen(new GameScreen(this));
    }
    
    @Override
    public void render() {
        Gdx.gl.glClearColor(0.05f, 0.05f, 0.2f, 1); // Dark space background
        Gdx.gl.glClear(GL20.GL_COLOR_BUFFER_BIT);
        super.render();
    }
    
    @Override
    public void dispose() {
        batch.dispose();
        assets.dispose();
    }
}
```

### 2. Assets Manager (Assets.java)
```java
package com.spaceclicker.game.utils;

import com.badlogic.gdx.assets.AssetManager;
import com.badlogic.gdx.audio.Sound;
import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.BitmapFont;
import com.badlogic.gdx.graphics.g2d.TextureAtlas;
import com.badlogic.gdx.graphics.g2d.freetype.FreeTypeFontGenerator;
import com.badlogic.gdx.Gdx;

public class Assets {
    public AssetManager manager;
    
    // Textures
    public Texture playerShip;
    public Texture enemyShip;
    public Texture background;
    public Texture laser;
    public Texture explosion;
    public Texture star;
    
    // Sounds
    public Sound laserSound;
    public Sound explosionSound;
    public Sound upgradeSound;
    public Sound backgroundMusic;
    
    // Fonts
    public BitmapFont font;
    public BitmapFont titleFont;
    
    public Assets() {
        manager = new AssetManager();
    }
    
    public void load() {
        // Load textures (you'll need to create these or use placeholder rectangles)
        manager.load("player_ship.png", Texture.class);
        manager.load("enemy_ship.png", Texture.class);
        manager.load("space_background.png", Texture.class);
        manager.load("laser.png", Texture.class);
        manager.load("explosion.png", Texture.class);
        manager.load("star.png", Texture.class);
        
        // Load sounds
        manager.load("laser.ogg", Sound.class);
        manager.load("explosion.ogg", Sound.class);
        manager.load("upgrade.ogg", Sound.class);
        
        // Create fonts
        createFonts();
    }
    
    private void createFonts() {
        FreeTypeFontGenerator generator = new FreeTypeFontGenerator(Gdx.files.internal("fonts/space_font.ttf"));
        FreeTypeFontGenerator.FreeTypeFontParameter parameter = new FreeTypeFontGenerator.FreeTypeFontParameter();
        
        parameter.size = 24;
        font = generator.generateFont(parameter);
        
        parameter.size = 48;
        titleFont = generator.generateFont(parameter);
        
        generator.dispose();
    }
    
    public void finishLoading() {
        manager.finishLoading();
        
        // Get loaded assets
        playerShip = manager.get("player_ship.png", Texture.class);
        enemyShip = manager.get("enemy_ship.png", Texture.class);
        background = manager.get("space_background.png", Texture.class);
        laser = manager.get("laser.png", Texture.class);
        explosion = manager.get("explosion.png", Texture.class);
        star = manager.get("star.png", Texture.class);
        
        laserSound = manager.get("laser.ogg", Sound.class);
        explosionSound = manager.get("explosion.ogg", Sound.class);
        upgradeSound = manager.get("upgrade.ogg", Sound.class);
    }
    
    public void dispose() {
        manager.dispose();
        font.dispose();
        titleFont.dispose();
    }
}
```

### 3. Player Entity (Player.java)
```java
package com.spaceclicker.game.entities;

import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.Vector2;

public class Player {
    private Vector2 position;
    private Texture texture;
    private float health;
    private float maxHealth;
    private float damage;
    private float speed;
    private float fireRate;
    private float lastFireTime;
    
    // Upgrade levels
    private int damageLevel;
    private int healthLevel;
    private int speedLevel;
    private int fireRateLevel;
    
    public Player(Texture texture, float x, float y) {
        this.texture = texture;
        this.position = new Vector2(x, y);
        
        // Base stats
        this.maxHealth = 100;
        this.health = maxHealth;
        this.damage = 10;
        this.speed = 200;
        this.fireRate = 0.5f; // shots per second
        
        // Upgrade levels
        this.damageLevel = 1;
        this.healthLevel = 1;
        this.speedLevel = 1;
        this.fireRateLevel = 1;
    }
    
    public void update(float delta) {
        lastFireTime += delta;
        
        // Auto-regenerate health slowly
        if (health < maxHealth) {
            health += 5 * delta;
            if (health > maxHealth) health = maxHealth;
        }
    }
    
    public void render(SpriteBatch batch) {
        batch.draw(texture, position.x, position.y);
    }
    
    public boolean canFire() {
        return lastFireTime >= (1f / fireRate);
    }
    
    public void fire() {
        if (canFire()) {
            lastFireTime = 0;
        }
    }
    
    public void takeDamage(float damage) {
        health -= damage;
        if (health < 0) health = 0;
    }
    
    public void upgradeHealth() {
        healthLevel++;
        float healthIncrease = 25 * healthLevel;
        maxHealth += healthIncrease;
        health += healthIncrease; // Also heal when upgrading
    }
    
    public void upgradeDamage() {
        damageLevel++;
        damage += 5 * damageLevel;
    }
    
    public void upgradeSpeed() {
        speedLevel++;
        speed += 50 * speedLevel;
    }
    
    public void upgradeFireRate() {
        fireRateLevel++;
        fireRate += 0.2f * fireRateLevel;
    }
    
    // Getters
    public Vector2 getPosition() { return position; }
    public float getHealth() { return health; }
    public float getMaxHealth() { return maxHealth; }
    public float getDamage() { return damage; }
    public float getSpeed() { return speed; }
    public int getDamageLevel() { return damageLevel; }
    public int getHealthLevel() { return healthLevel; }
    public int getSpeedLevel() { return speedLevel; }
    public int getFireRateLevel() { return fireRateLevel; }
}
```

### 4. Enemy Entity (Enemy.java)
```java
package com.spaceclicker.game.entities;

import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.Vector2;
import com.badlogic.gdx.math.MathUtils;

public class Enemy {
    public enum EnemyType {
        SCOUT(50, 5, 100, 10),
        FIGHTER(100, 15, 80, 25),
        CRUISER(200, 30, 60, 50),
        BATTLESHIP(500, 60, 40, 100);
        
        public final float health;
        public final float damage;
        public final float speed;
        public final int reward;
        
        EnemyType(float health, float damage, float speed, int reward) {
            this.health = health;
            this.damage = damage;
            this.speed = speed;
            this.reward = reward;
        }
    }
    
    private Vector2 position;
    private Vector2 velocity;
    private Texture texture;
    private EnemyType type;
    private float health;
    private float maxHealth;
    private boolean destroyed;
    private float attackCooldown;
    private float scale;
    
    public Enemy(Texture texture, EnemyType type, float x, float y) {
        this.texture = texture;
        this.type = type;
        this.position = new Vector2(x, y);
        this.velocity = new Vector2(-type.speed, MathUtils.random(-50, 50));
        this.health = type.health;
        this.maxHealth = type.health;
        this.destroyed = false;
        this.attackCooldown = 0;
        this.scale = getScaleForType(type);
    }
    
    private float getScaleForType(EnemyType type) {
        switch(type) {
            case SCOUT: return 0.8f;
            case FIGHTER: return 1.0f;
            case CRUISER: return 1.3f;
            case BATTLESHIP: return 1.6f;
            default: return 1.0f;
        }
    }
    
    public void update(float delta) {
        if (destroyed) return;
        
        // Move enemy
        position.add(velocity.x * delta, velocity.y * delta);
        
        // Update attack cooldown
        attackCooldown -= delta;
        
        // Destroy if off-screen
        if (position.x < -100) {
            destroyed = true;
        }
    }
    
    public void render(SpriteBatch batch) {
        if (destroyed) return;
        
        // Draw enemy with scaling
        batch.draw(texture, position.x, position.y, 
                  texture.getWidth() * scale, texture.getHeight() * scale);
        
        // Draw health bar
        drawHealthBar(batch);
    }
    
    private void drawHealthBar(SpriteBatch batch) {
        float healthPercent = health / maxHealth;
        float barWidth = texture.getWidth() * scale;
        float barHeight = 5;
        
        // Background (red)
        batch.setColor(1, 0, 0, 0.8f);
        batch.draw(texture, position.x, position.y + texture.getHeight() * scale + 5, 
                  barWidth, barHeight);
        
        // Foreground (green)
        batch.setColor(0, 1, 0, 0.8f);
        batch.draw(texture, position.x, position.y + texture.getHeight() * scale + 5, 
                  barWidth * healthPercent, barHeight);
        
        batch.setColor(1, 1, 1, 1); // Reset color
    }
    
    public void takeDamage(float damage) {
        health -= damage;
        if (health <= 0) {
            destroyed = true;
        }
    }
    
    public boolean canAttack() {
        return attackCooldown <= 0;
    }
    
    public void attack() {
        attackCooldown = 2.0f; // 2 second cooldown
    }
    
    // Getters
    public Vector2 getPosition() { return position; }
    public EnemyType getType() { return type; }
    public boolean isDestroyed() { return destroyed; }
    public float getHealth() { return health; }
    public float getMaxHealth() { return maxHealth; }
    public Texture getTexture() { return texture; }
    public float getScale() { return scale; }
}
```

I'll continue with the rest of the implementation in the next part...