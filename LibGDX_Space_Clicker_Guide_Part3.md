### 11. Upgrade UI (UpgradeUI.java)
```java
package com.spaceclicker.game.ui;

import com.badlogic.gdx.graphics.g2d.BitmapFont;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.graphics.glutils.ShapeRenderer;
import com.spaceclicker.game.entities.Player;
import com.spaceclicker.game.systems.UpgradeSystem;
import com.spaceclicker.game.utils.GameData;

public class UpgradeUI {
    private BitmapFont font;
    private BitmapFont titleFont;
    private UpgradeSystem upgradeSystem;
    private GameData gameData;
    private ShapeRenderer shapeRenderer;
    private boolean visible;
    
    public UpgradeUI(BitmapFont font, BitmapFont titleFont, UpgradeSystem upgradeSystem, GameData gameData) {
        this.font = font;
        this.titleFont = titleFont;
        this.upgradeSystem = upgradeSystem;
        this.gameData = gameData;
        this.shapeRenderer = new ShapeRenderer();
        this.visible = false;
    }
    
    public void render(SpriteBatch batch, Player player) {
        if (!visible) return;
        
        // Draw background
        batch.end();
        shapeRenderer.begin(ShapeRenderer.ShapeType.Filled);
        shapeRenderer.setColor(0, 0, 0, 0.8f);
        shapeRenderer.rect(200, 100, 400, 400);
        shapeRenderer.setColor(0.2f, 0.2f, 0.8f, 1);
        shapeRenderer.rect(205, 105, 390, 390);
        shapeRenderer.end();
        batch.begin();
        
        // Title
        titleFont.draw(batch, "UPGRADES", 320, 470);
        
        // Money
        font.draw(batch, "Money: $" + gameData.getMoney(), 220, 440);
        
        // Upgrade options
        int y = 400;
        
        // Damage upgrade
        int damageCost = upgradeSystem.getUpgradeCost("damage", player.getDamageLevel());
        font.draw(batch, "Damage Level " + player.getDamageLevel(), 220, y);
        if (upgradeSystem.canAfford("damage", player.getDamageLevel())) {
            font.draw(batch, "Press 1 - $" + damageCost, 220, y - 25);
        } else {
            font.draw(batch, "Need $" + damageCost, 220, y - 25);
        }
        y -= 70;
        
        // Health upgrade
        int healthCost = upgradeSystem.getUpgradeCost("health", player.getHealthLevel());
        font.draw(batch, "Health Level " + player.getHealthLevel(), 220, y);
        if (upgradeSystem.canAfford("health", player.getHealthLevel())) {
            font.draw(batch, "Press 2 - $" + healthCost, 220, y - 25);
        } else {
            font.draw(batch, "Need $" + healthCost, 220, y - 25);
        }
        y -= 70;
        
        // Speed upgrade
        int speedCost = upgradeSystem.getUpgradeCost("speed", player.getSpeedLevel());
        font.draw(batch, "Speed Level " + player.getSpeedLevel(), 220, y);
        if (upgradeSystem.canAfford("speed", player.getSpeedLevel())) {
            font.draw(batch, "Press 3 - $" + speedCost, 220, y - 25);
        } else {
            font.draw(batch, "Need $" + speedCost, 220, y - 25);
        }
        y -= 70;
        
        // Fire rate upgrade
        int fireRateCost = upgradeSystem.getUpgradeCost("firerate", player.getFireRateLevel());
        font.draw(batch, "Fire Rate Level " + player.getFireRateLevel(), 220, y);
        if (upgradeSystem.canAfford("firerate", player.getFireRateLevel())) {
            font.draw(batch, "Press 4 - $" + fireRateCost, 220, y - 25);
        } else {
            font.draw(batch, "Need $" + fireRateCost, 220, y - 25);
        }
        
        // Instructions
        font.draw(batch, "Press U to close", 220, 140);
    }
    
    public void toggle() {
        visible = !visible;
    }
    
    public boolean isVisible() {
        return visible;
    }
    
    public void dispose() {
        shapeRenderer.dispose();
    }
}
```

### 12. Main Game Screen (GameScreen.java)
```java
package com.spaceclicker.game;

import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Input;
import com.badlogic.gdx.Screen;
import com.badlogic.gdx.audio.Sound;
import com.badlogic.gdx.graphics.GL20;
import com.badlogic.gdx.graphics.OrthographicCamera;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.utils.viewport.FitViewport;
import com.badlogic.gdx.utils.viewport.Viewport;
import com.spaceclicker.game.entities.Enemy;
import com.spaceclicker.game.entities.Player;
import com.spaceclicker.game.entities.Projectile;
import com.spaceclicker.game.systems.EnemySpawner;
import com.spaceclicker.game.systems.ParticleManager;
import com.spaceclicker.game.systems.UpgradeSystem;
import com.spaceclicker.game.ui.GameUI;
import com.spaceclicker.game.ui.UpgradeUI;
import com.spaceclicker.game.utils.GameData;

import java.util.ArrayList;
import java.util.List;

public class GameScreen implements Screen {
    private SpaceClickerGame game;
    private OrthographicCamera camera;
    private Viewport viewport;
    private SpriteBatch batch;
    
    // Game objects
    private Player player;
    private EnemySpawner enemySpawner;
    private List<Projectile> projectiles;
    private ParticleManager particleManager;
    private UpgradeSystem upgradeSystem;
    private GameData gameData;
    
    // UI
    private GameUI gameUI;
    private UpgradeUI upgradeUI;
    
    // Constants
    private static final int WORLD_WIDTH = 800;
    private static final int WORLD_HEIGHT = 600;
    
    public GameScreen(SpaceClickerGame game) {
        this.game = game;
        this.batch = game.batch;
        
        // Create camera and viewport
        camera = new OrthographicCamera();
        viewport = new FitViewport(WORLD_WIDTH, WORLD_HEIGHT, camera);
        
        // Initialize game systems
        gameData = new GameData();
        upgradeSystem = new UpgradeSystem(gameData);
        
        // Initialize game objects
        player = new Player(game.assets.playerShip, 50, WORLD_HEIGHT / 2);
        enemySpawner = new EnemySpawner(game.assets.enemyShip, gameData);
        projectiles = new ArrayList<>();
        particleManager = new ParticleManager(game.assets.star, game.assets.explosion);
        
        // Initialize UI
        gameUI = new GameUI(game.assets.font, game.assets.titleFont, gameData);
        upgradeUI = new UpgradeUI(game.assets.font, game.assets.titleFont, upgradeSystem, gameData);
    }
    
    @Override
    public void render(float delta) {
        // Clear screen
        Gdx.gl.glClearColor(0.05f, 0.05f, 0.2f, 1);
        Gdx.gl.glClear(GL20.GL_COLOR_BUFFER_BIT);
        
        // Update game logic
        update(delta);
        
        // Set camera and begin batch
        camera.update();
        batch.setProjectionMatrix(camera.combined);
        batch.begin();
        
        // Draw background
        batch.draw(game.assets.background, 0, 0, WORLD_WIDTH, WORLD_HEIGHT);
        
        // Draw particles (background)
        particleManager.render(batch);
        
        // Draw player
        player.render(batch);
        
        // Draw enemies
        for (Enemy enemy : enemySpawner.getEnemies()) {
            enemy.render(batch);
        }
        
        // Draw projectiles
        for (Projectile projectile : projectiles) {
            projectile.render(batch);
        }
        
        // Draw UI
        gameUI.render(batch, player, enemySpawner);
        upgradeUI.render(batch, player);
        
        batch.end();
    }
    
    private void update(float delta) {
        // Handle input
        handleInput();
        
        // Update game objects
        player.update(delta);
        enemySpawner.update(delta);
        particleManager.update(delta);
        
        // Update projectiles
        for (int i = projectiles.size() - 1; i >= 0; i--) {
            Projectile projectile = projectiles.get(i);
            projectile.update(delta);
            
            if (projectile.isDestroyed()) {
                projectiles.remove(i);
            }
        }
        
        // Check collisions
        checkCollisions();
        
        // Enemy AI - attack player
        enemyAI();
    }
    
    private void handleInput() {
        // Click to shoot
        if (Gdx.input.justTouched()) {
            fireProjectile();
        }
        
        // Upgrade menu
        if (Gdx.input.isKeyJustPressed(Input.Keys.U)) {
            upgradeUI.toggle();
        }
        
        // Handle upgrades
        if (upgradeUI.isVisible()) {
            if (Gdx.input.isKeyJustPressed(Input.Keys.NUM_1)) {
                if (upgradeSystem.purchaseUpgrade("damage", player)) {
                    game.assets.upgradeSound.play(0.5f);
                }
            }
            if (Gdx.input.isKeyJustPressed(Input.Keys.NUM_2)) {
                if (upgradeSystem.purchaseUpgrade("health", player)) {
                    game.assets.upgradeSound.play(0.5f);
                }
            }
            if (Gdx.input.isKeyJustPressed(Input.Keys.NUM_3)) {
                if (upgradeSystem.purchaseUpgrade("speed", player)) {
                    game.assets.upgradeSound.play(0.5f);
                }
            }
            if (Gdx.input.isKeyJustPressed(Input.Keys.NUM_4)) {
                if (upgradeSystem.purchaseUpgrade("firerate", player)) {
                    game.assets.upgradeSound.play(0.5f);
                }
            }
        }
        
        // Reset save data
        if (Gdx.input.isKeyJustPressed(Input.Keys.R)) {
            gameData.resetData();
        }
        
        // Exit game
        if (Gdx.input.isKeyJustPressed(Input.Keys.ESCAPE)) {
            Gdx.app.exit();
        }
    }
    
    private void fireProjectile() {
        if (player.canFire()) {
            player.fire();
            gameData.incrementClickCount();
            
            // Create projectile
            float x = player.getPosition().x + 60; // Offset from player
            float y = player.getPosition().y + 30;
            Projectile projectile = new Projectile(game.assets.laser, x, y, player.getDamage());
            projectiles.add(projectile);
            
            // Play sound
            game.assets.laserSound.play(0.3f);
        }
    }
    
    private void checkCollisions() {
        // Projectile vs Enemy collision
        for (int i = projectiles.size() - 1; i >= 0; i--) {
            Projectile projectile = projectiles.get(i);
            
            for (Enemy enemy : enemySpawner.getEnemies()) {
                if (projectile.checkCollision(enemy)) {
                    enemy.takeDamage(projectile.getDamage());
                    projectile.destroy();
                    
                    // Create explosion effect
                    particleManager.createExplosion(enemy.getPosition().x, enemy.getPosition().y);
                    
                    if (enemy.isDestroyed()) {
                        gameData.addMoney(enemy.getType().reward);
                        gameData.incrementEnemiesKilled();
                        game.assets.explosionSound.play(0.4f);
                    }
                    
                    break;
                }
            }
        }
    }
    
    private void enemyAI() {
        for (Enemy enemy : enemySpawner.getEnemies()) {
            // Check if enemy is close enough to attack
            float distance = enemy.getPosition().dst(player.getPosition());
            if (distance < 100 && enemy.canAttack()) {
                enemy.attack();
                player.takeDamage(enemy.getType().damage);
                
                // Create damage effect
                particleManager.createExplosion(player.getPosition().x, player.getPosition().y);
            }
        }
    }
    
    @Override
    public void resize(int width, int height) {
        viewport.update(width, height);
    }
    
    @Override
    public void show() {}
    
    @Override
    public void hide() {}
    
    @Override
    public void pause() {}
    
    @Override
    public void resume() {}
    
    @Override
    public void dispose() {
        upgradeUI.dispose();
    }
}
```

### 13. Desktop Launcher (DesktopLauncher.java)
```java
package com.spaceclicker.game.desktop;

import com.badlogic.gdx.backends.lwjgl.LwjglApplication;
import com.badlogic.gdx.backends.lwjgl.LwjglApplicationConfiguration;
import com.spaceclicker.game.SpaceClickerGame;

public class DesktopLauncher {
    public static void main (String[] arg) {
        LwjglApplicationConfiguration config = new LwjglApplicationConfiguration();
        config.title = "Space Clicker";
        config.width = 800;
        config.height = 600;
        config.resizable = false;
        
        new LwjglApplication(new SpaceClickerGame(), config);
    }
}
```

## 🎨 Asset Creation Tips

### Creating Basic Assets (if you don't have graphics skills):
1. **Use colored rectangles** for prototyping
2. **Online tools**: Piskel, Aseprite for pixel art
3. **Free assets**: OpenGameArt.org, Kenney.nl
4. **AI generation**: Use DALL-E, Midjourney for concept art

### Sound Assets:
- **Freesound.org**: Free sound effects
- **Zapsplat**: Professional sounds (free with registration)
- **Audacity**: Free audio editor

## 🚀 Special Features Implemented

1. **Progressive Difficulty**: Enemy spawn rate increases over time
2. **Multiple Enemy Types**: Scout, Fighter, Cruiser, Battleship
3. **Upgrade System**: Damage, Health, Speed, Fire Rate
4. **Particle Effects**: Explosions, background stars
5. **Save System**: Persistent progress
6. **Sound Effects**: Laser, explosion, upgrade sounds
7. **Visual Feedback**: Health bars, damage numbers
8. **Wave System**: Increasing difficulty with waves
9. **Statistics Tracking**: Kills, clicks, money earned
10. **Smooth Animations**: Projectile movement, enemy movement

## 📱 Building and Running

1. **Setup LibGDX project** using gdx-setup.jar
2. **Copy the code** into appropriate files
3. **Add placeholder textures** (colored rectangles work initially)
4. **Run from desktop launcher**
5. **Iterate and improve** based on gameplay feel

## 🔧 Expansion Ideas

- **Boss battles** every 10 waves
- **Power-ups** that drop from enemies
- **Different weapons** (missiles, lasers, plasma)
- **Shield system** for player
- **Achievements** system
- **Multiplayer** competition
- **Different ship types** to unlock
- **Special abilities** with cooldowns

This implementation provides a solid foundation for a space clicker game with room for expansion!