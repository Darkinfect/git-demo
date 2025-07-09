### 5. Upgrade System (UpgradeSystem.java)
```java
package com.spaceclicker.game.systems;

import com.spaceclicker.game.entities.Player;
import com.spaceclicker.game.utils.GameData;

public class UpgradeSystem {
    private GameData gameData;
    
    public UpgradeSystem(GameData gameData) {
        this.gameData = gameData;
    }
    
    public int getUpgradeCost(String upgradeType, int currentLevel) {
        int baseCost = getBaseCost(upgradeType);
        return (int)(baseCost * Math.pow(1.5, currentLevel - 1));
    }
    
    private int getBaseCost(String upgradeType) {
        switch(upgradeType) {
            case "damage": return 100;
            case "health": return 150;
            case "speed": return 200;
            case "firerate": return 300;
            default: return 100;
        }
    }
    
    public boolean canAfford(String upgradeType, int currentLevel) {
        int cost = getUpgradeCost(upgradeType, currentLevel);
        return gameData.getMoney() >= cost;
    }
    
    public boolean purchaseUpgrade(String upgradeType, Player player) {
        int currentLevel = getCurrentLevel(upgradeType, player);
        int cost = getUpgradeCost(upgradeType, currentLevel);
        
        if (gameData.getMoney() >= cost) {
            gameData.spendMoney(cost);
            applyUpgrade(upgradeType, player);
            return true;
        }
        return false;
    }
    
    private int getCurrentLevel(String upgradeType, Player player) {
        switch(upgradeType) {
            case "damage": return player.getDamageLevel();
            case "health": return player.getHealthLevel();
            case "speed": return player.getSpeedLevel();
            case "firerate": return player.getFireRateLevel();
            default: return 1;
        }
    }
    
    private void applyUpgrade(String upgradeType, Player player) {
        switch(upgradeType) {
            case "damage":
                player.upgradeDamage();
                break;
            case "health":
                player.upgradeHealth();
                break;
            case "speed":
                player.upgradeSpeed();
                break;
            case "firerate":
                player.upgradeFireRate();
                break;
        }
    }
}
```

### 6. Enemy Spawner (EnemySpawner.java)
```java
package com.spaceclicker.game.systems;

import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.math.MathUtils;
import com.spaceclicker.game.entities.Enemy;
import com.spaceclicker.game.utils.GameData;

import java.util.ArrayList;
import java.util.List;

public class EnemySpawner {
    private List<Enemy> enemies;
    private Texture enemyTexture;
    private float spawnTimer;
    private float spawnInterval;
    private GameData gameData;
    private int wave;
    
    public EnemySpawner(Texture enemyTexture, GameData gameData) {
        this.enemyTexture = enemyTexture;
        this.gameData = gameData;
        this.enemies = new ArrayList<>();
        this.spawnInterval = 2.0f;
        this.spawnTimer = 0;
        this.wave = 1;
    }
    
    public void update(float delta) {
        spawnTimer += delta;
        
        if (spawnTimer >= spawnInterval) {
            spawnEnemy();
            spawnTimer = 0;
            
            // Increase difficulty over time
            if (gameData.getEnemiesKilled() > 0 && gameData.getEnemiesKilled() % 10 == 0) {
                wave++;
                spawnInterval = Math.max(0.5f, spawnInterval - 0.1f);
            }
        }
        
        // Update all enemies
        for (int i = enemies.size() - 1; i >= 0; i--) {
            Enemy enemy = enemies.get(i);
            enemy.update(delta);
            
            if (enemy.isDestroyed()) {
                enemies.remove(i);
            }
        }
    }
    
    private void spawnEnemy() {
        float screenWidth = 800; // Adjust based on your screen width
        float screenHeight = 600; // Adjust based on your screen height
        
        // Spawn at random Y position on the right side
        float y = MathUtils.random(50, screenHeight - 100);
        
        // Choose enemy type based on wave
        Enemy.EnemyType type = chooseEnemyType();
        
        Enemy enemy = new Enemy(enemyTexture, type, screenWidth, y);
        enemies.add(enemy);
    }
    
    private Enemy.EnemyType chooseEnemyType() {
        float random = MathUtils.random();
        
        if (wave < 3) {
            return Enemy.EnemyType.SCOUT;
        } else if (wave < 6) {
            return random < 0.7f ? Enemy.EnemyType.SCOUT : Enemy.EnemyType.FIGHTER;
        } else if (wave < 10) {
            if (random < 0.5f) return Enemy.EnemyType.SCOUT;
            else if (random < 0.8f) return Enemy.EnemyType.FIGHTER;
            else return Enemy.EnemyType.CRUISER;
        } else {
            if (random < 0.3f) return Enemy.EnemyType.SCOUT;
            else if (random < 0.6f) return Enemy.EnemyType.FIGHTER;
            else if (random < 0.85f) return Enemy.EnemyType.CRUISER;
            else return Enemy.EnemyType.BATTLESHIP;
        }
    }
    
    public List<Enemy> getEnemies() {
        return enemies;
    }
    
    public int getWave() {
        return wave;
    }
}
```

### 7. Projectile System (Projectile.java)
```java
package com.spaceclicker.game.entities;

import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.Vector2;

public class Projectile {
    private Vector2 position;
    private Vector2 velocity;
    private Texture texture;
    private float damage;
    private boolean destroyed;
    private float lifetime;
    private float maxLifetime;
    
    public Projectile(Texture texture, float x, float y, float damage) {
        this.texture = texture;
        this.position = new Vector2(x, y);
        this.velocity = new Vector2(400, 0); // Move right
        this.damage = damage;
        this.destroyed = false;
        this.lifetime = 0;
        this.maxLifetime = 3.0f; // 3 seconds max
    }
    
    public void update(float delta) {
        if (destroyed) return;
        
        position.add(velocity.x * delta, velocity.y * delta);
        lifetime += delta;
        
        // Destroy if too old or off-screen
        if (lifetime >= maxLifetime || position.x > 900) {
            destroyed = true;
        }
    }
    
    public void render(SpriteBatch batch) {
        if (!destroyed) {
            batch.draw(texture, position.x, position.y);
        }
    }
    
    public boolean checkCollision(Enemy enemy) {
        if (destroyed || enemy.isDestroyed()) return false;
        
        Vector2 enemyPos = enemy.getPosition();
        float enemySize = enemy.getTexture().getWidth() * enemy.getScale();
        
        return position.x < enemyPos.x + enemySize &&
               position.x + texture.getWidth() > enemyPos.x &&
               position.y < enemyPos.y + enemySize &&
               position.y + texture.getHeight() > enemyPos.y;
    }
    
    public void destroy() {
        destroyed = true;
    }
    
    // Getters
    public Vector2 getPosition() { return position; }
    public float getDamage() { return damage; }
    public boolean isDestroyed() { return destroyed; }
}
```

### 8. Particle Manager (ParticleManager.java)
```java
package com.spaceclicker.game.systems;

import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.MathUtils;
import com.badlogic.gdx.math.Vector2;

import java.util.ArrayList;
import java.util.List;

public class ParticleManager {
    private List<Particle> particles;
    private Texture starTexture;
    private Texture explosionTexture;
    
    public ParticleManager(Texture starTexture, Texture explosionTexture) {
        this.particles = new ArrayList<>();
        this.starTexture = starTexture;
        this.explosionTexture = explosionTexture;
        
        // Create background stars
        createBackgroundStars();
    }
    
    private void createBackgroundStars() {
        for (int i = 0; i < 100; i++) {
            float x = MathUtils.random(0, 800);
            float y = MathUtils.random(0, 600);
            float speed = MathUtils.random(10, 50);
            particles.add(new Particle(starTexture, x, y, -speed, 0, 10.0f, 0.3f));
        }
    }
    
    public void update(float delta) {
        for (int i = particles.size() - 1; i >= 0; i--) {
            Particle particle = particles.get(i);
            particle.update(delta);
            
            if (particle.isDestroyed()) {
                particles.remove(i);
            }
        }
        
        // Add new background stars
        if (MathUtils.random() < 0.1f) {
            float y = MathUtils.random(0, 600);
            float speed = MathUtils.random(10, 50);
            particles.add(new Particle(starTexture, 800, y, -speed, 0, 10.0f, 0.3f));
        }
    }
    
    public void render(SpriteBatch batch) {
        for (Particle particle : particles) {
            particle.render(batch);
        }
    }
    
    public void createExplosion(float x, float y) {
        for (int i = 0; i < 20; i++) {
            float velX = MathUtils.random(-200, 200);
            float velY = MathUtils.random(-200, 200);
            particles.add(new Particle(explosionTexture, x, y, velX, velY, 1.0f, 0.8f));
        }
    }
    
    private static class Particle {
        private Vector2 position;
        private Vector2 velocity;
        private Texture texture;
        private float lifetime;
        private float maxLifetime;
        private float alpha;
        private boolean destroyed;
        
        public Particle(Texture texture, float x, float y, float velX, float velY, float lifetime, float alpha) {
            this.texture = texture;
            this.position = new Vector2(x, y);
            this.velocity = new Vector2(velX, velY);
            this.lifetime = 0;
            this.maxLifetime = lifetime;
            this.alpha = alpha;
            this.destroyed = false;
        }
        
        public void update(float delta) {
            position.add(velocity.x * delta, velocity.y * delta);
            lifetime += delta;
            
            // Fade out over time
            alpha = 1.0f - (lifetime / maxLifetime);
            
            if (lifetime >= maxLifetime || position.x < -50) {
                destroyed = true;
            }
        }
        
        public void render(SpriteBatch batch) {
            batch.setColor(1, 1, 1, alpha);
            batch.draw(texture, position.x, position.y);
            batch.setColor(1, 1, 1, 1);
        }
        
        public boolean isDestroyed() {
            return destroyed;
        }
    }
}
```

### 9. Game Data (GameData.java)
```java
package com.spaceclicker.game.utils;

import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Preferences;

public class GameData {
    private int money;
    private int totalMoney;
    private int enemiesKilled;
    private int clickCount;
    private Preferences prefs;
    
    public GameData() {
        prefs = Gdx.app.getPreferences("SpaceClickerSave");
        loadData();
    }
    
    public void addMoney(int amount) {
        money += amount;
        totalMoney += amount;
        saveData();
    }
    
    public void spendMoney(int amount) {
        money -= amount;
        saveData();
    }
    
    public void incrementEnemiesKilled() {
        enemiesKilled++;
        saveData();
    }
    
    public void incrementClickCount() {
        clickCount++;
        saveData();
    }
    
    public void saveData() {
        prefs.putInteger("money", money);
        prefs.putInteger("totalMoney", totalMoney);
        prefs.putInteger("enemiesKilled", enemiesKilled);
        prefs.putInteger("clickCount", clickCount);
        prefs.flush();
    }
    
    public void loadData() {
        money = prefs.getInteger("money", 0);
        totalMoney = prefs.getInteger("totalMoney", 0);
        enemiesKilled = prefs.getInteger("enemiesKilled", 0);
        clickCount = prefs.getInteger("clickCount", 0);
    }
    
    public void resetData() {
        money = 0;
        totalMoney = 0;
        enemiesKilled = 0;
        clickCount = 0;
        saveData();
    }
    
    // Getters
    public int getMoney() { return money; }
    public int getTotalMoney() { return totalMoney; }
    public int getEnemiesKilled() { return enemiesKilled; }
    public int getClickCount() { return clickCount; }
}
```

### 10. Game UI (GameUI.java)
```java
package com.spaceclicker.game.ui;

import com.badlogic.gdx.graphics.g2d.BitmapFont;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.spaceclicker.game.entities.Player;
import com.spaceclicker.game.systems.EnemySpawner;
import com.spaceclicker.game.utils.GameData;

public class GameUI {
    private BitmapFont font;
    private BitmapFont titleFont;
    private GameData gameData;
    
    public GameUI(BitmapFont font, BitmapFont titleFont, GameData gameData) {
        this.font = font;
        this.titleFont = titleFont;
        this.gameData = gameData;
    }
    
    public void render(SpriteBatch batch, Player player, EnemySpawner spawner) {
        // Title
        titleFont.draw(batch, "SPACE CLICKER", 20, 580);
        
        // Player stats
        font.draw(batch, "Money: $" + gameData.getMoney(), 20, 540);
        font.draw(batch, "Health: " + (int)player.getHealth() + "/" + (int)player.getMaxHealth(), 20, 510);
        font.draw(batch, "Damage: " + (int)player.getDamage(), 20, 480);
        font.draw(batch, "Speed: " + (int)player.getSpeed(), 20, 450);
        
        // Game stats
        font.draw(batch, "Wave: " + spawner.getWave(), 20, 400);
        font.draw(batch, "Enemies Killed: " + gameData.getEnemiesKilled(), 20, 370);
        font.draw(batch, "Total Clicks: " + gameData.getClickCount(), 20, 340);
        
        // Instructions
        font.draw(batch, "Click to shoot!", 20, 280);
        font.draw(batch, "Press U for upgrades", 20, 250);
        font.draw(batch, "Press R to reset save", 20, 220);
    }
}
```

Continue to Part 3...