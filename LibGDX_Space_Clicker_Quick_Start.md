# 🎮 LibGDX Space Clicker - Quick Start Guide

## 🚀 Getting Started (Step-by-Step)

### 1. Download LibGDX Setup Tool
- Go to https://libgdx.com/dev/setup/
- Download the setup tool (gdx-setup.jar)
- Run it: `java -jar gdx-setup.jar`

### 2. Project Configuration
```
Name: SpaceClicker
Package: com.spaceclicker.game
Game class: SpaceClickerGame
Destination: [Your chosen folder]
Android SDK: [Leave empty if desktop only]

Sub Projects:
✓ Desktop
☐ Android (unless you want mobile)
☐ iOS
☐ HTML

Extensions:
✓ Box2D
✓ Freetype
```

### 3. Initial File Structure After Setup
```
SpaceClicker/
├── build.gradle
├── settings.gradle
├── core/
│   ├── build.gradle
│   └── src/
│       └── com/spaceclicker/game/
│           └── SpaceClickerGame.java
├── desktop/
│   ├── build.gradle
│   └── src/
│       └── com/spaceclicker/game/desktop/
│           └── DesktopLauncher.java
└── assets/
    └── badlogic.jpg
```

### 4. Replace Generated Files
- Replace `SpaceClickerGame.java` with the code from Part 1
- Replace `DesktopLauncher.java` with the code from Part 3
- Create all the additional files from the guide

### 5. Add Required Assets
Create these placeholder files in the `assets/` folder:

**For Images (use simple colored rectangles initially):**
```
assets/
├── player_ship.png (32x32 blue rectangle)
├── enemy_ship.png (32x32 red rectangle)
├── laser.png (16x4 yellow rectangle)
├── explosion.png (8x8 orange square)
├── star.png (2x2 white square)
├── space_background.png (800x600 dark blue/black)
└── fonts/
    └── space_font.ttf (download any sci-fi font)
```

**For Sounds (optional for testing):**
```
assets/
├── laser.ogg
├── explosion.ogg
└── upgrade.ogg
```

### 6. Build and Run
```bash
# Navigate to project folder
cd SpaceClicker

# Run desktop version
./gradlew desktop:run

# Or on Windows:
gradlew.bat desktop:run
```

## 🎯 Game Controls

- **Click**: Shoot laser at enemies
- **U**: Open/close upgrade menu
- **1-4**: Purchase upgrades (when upgrade menu is open)
- **R**: Reset save data
- **ESC**: Exit game

## 🛠️ Creating Placeholder Assets

### Simple Image Creation (if you don't have graphics):

**Player Ship (player_ship.png):**
- 32x32 blue rectangle
- Can be created in MS Paint or any image editor

**Enemy Ship (enemy_ship.png):**
- 32x32 red rectangle

**Laser (laser.png):**
- 16x4 yellow rectangle

**Explosion (explosion.png):**
- 8x8 orange square

**Star (star.png):**
- 2x2 white square

**Background (space_background.png):**
- 800x600 dark blue or black image

### Using Online Tools:
- **Piskel**: https://www.piskelapp.com/ (pixel art)
- **Canva**: https://www.canva.com/ (simple graphics)
- **GIMP**: https://www.gimp.org/ (free image editor)

## 🔧 Common Issues and Solutions

### Issue: "Could not find or load main class"
**Solution**: Make sure you're running from the correct directory and Java is installed.

### Issue: Assets not loading
**Solution**: 
1. Check that asset files are in the `assets/` folder
2. Make sure file names match exactly (case-sensitive)
3. For testing, create simple colored rectangles as placeholders

### Issue: Build errors
**Solution**:
1. Make sure all import statements are correct
2. Check that LibGDX version is compatible
3. Run `./gradlew clean` then `./gradlew desktop:run`

### Issue: Game crashes on startup
**Solution**:
1. Check console output for error messages
2. Make sure all required assets exist
3. Start with placeholder rectangular images

## 🎨 Enhancing Your Game

### Phase 1: Basic Functionality
- Get the basic game running with placeholder graphics
- Test clicking, shooting, and upgrades
- Verify save/load system works

### Phase 2: Better Graphics
- Replace placeholder rectangles with actual sprite art
- Add animations for shooting and explosions
- Improve UI design

### Phase 3: Advanced Features
- Add sound effects and music
- Implement more enemy types
- Add boss battles
- Create particle effects for explosions

### Phase 4: Polish
- Add screen transitions
- Implement achievement system
- Create more upgrade types
- Add different weapons

## 📚 Learning Resources

### LibGDX Documentation:
- **Official Wiki**: https://libgdx.com/wiki/
- **API Documentation**: https://libgdx.badlogicgames.com/ci/nightlies/docs/api/

### Tutorials:
- **LibGDX Tutorial Series**: https://www.youtube.com/watch?v=mmH0sSg4_rY
- **Game Development with LibGDX**: https://www.gamefromscratch.com/page/LibGDX-Tutorial-series.aspx

### Asset Resources:
- **Free Sprites**: https://opengameart.org/
- **Kenney Assets**: https://www.kenney.nl/assets
- **Free Sounds**: https://freesound.org/

## 🚀 Next Steps

1. **Get it running**: Start with placeholder graphics
2. **Test core mechanics**: Make sure clicking and upgrades work
3. **Add real graphics**: Replace placeholders with actual sprites
4. **Enhance gameplay**: Add more enemy types and features
5. **Polish**: Add sound, effects, and juice

Remember: Start simple and iterate! The most important thing is getting a working game, then you can enhance it step by step.

Happy coding! 🎮✨