# Interactive Waterfall - Hand Tracking Installation

An interactive browser-based art installation that uses webcam hand tracking to create a dynamic waterfall effect. Move your hands in front of the camera to part and bend the flowing water particles.

## Features

- **Real-time hand tracking** using MediaPipe Hands (detects up to 2 hands)
- **2000 animated particles** creating a flowing waterfall effect
- **Physics-based interaction** - particles avoid your hands with natural flowing motion
- **Customizable parameters** - easily adjust particle count, forces, colors, and more
- **No camera feed shown** - only the stylized waterfall visualization
- **Full-screen canvas** with optional motion blur trails

## How to Run Locally

### Option 1: Using a Local Web Server (Recommended)

The project requires a web server due to browser security restrictions on camera access.

**Using Python:**
```bash
# Python 3
python -m http.server 8000

# Python 2
python -m SimpleHTTPServer 8000
```

**Using Node.js (http-server):**
```bash
# Install http-server globally (once)
npm install -g http-server

# Run server
http-server -p 8000
```

**Using PHP:**
```bash
php -S localhost:8000
```

Then open your browser and navigate to:
```
http://localhost:8000
```

### Option 2: Using VS Code Live Server Extension

1. Install the "Live Server" extension in VS Code
2. Right-click on `index.html`
3. Select "Open with Live Server"

### Option 3: Direct File Access (May not work)

Some browsers may block camera access when opening files directly with `file://` protocol. If this works for you:
```
Open index.html directly in your browser
```

## Browser Requirements

- **Modern browser** with WebGL support (Chrome, Firefox, Edge, Safari)
- **Camera access** - you must allow the browser to access your webcam
- **HTTPS or localhost** - browsers require secure context for camera access

## First Time Setup

1. Open the page in your browser
2. **Allow camera access** when prompted
3. Wait for "Loading hand tracking..." message to disappear
4. Move your hands in front of the camera
5. Watch the waterfall particles flow around your hands!

## Customization

All parameters can be adjusted at the top of the `<script>` section in `index.html`. Look for the `CONFIG` object:

### Particle System
```javascript
PARTICLE_COUNT: 2000,           // More particles = denser waterfall
PARTICLE_SIZE_MIN: 1,           // Minimum particle size (pixels)
PARTICLE_SIZE_MAX: 3,           // Maximum particle size (pixels)
PARTICLE_SPEED_MIN: 2,          // Initial fall speed range
PARTICLE_SPEED_MAX: 5,
```

### Physics
```javascript
GRAVITY: 0.15,                  // Higher = faster falling
VELOCITY_DAMPING: 0.98,         // Lower = more fluid motion
MAX_VELOCITY: 15,               // Prevents particles from going too fast
```

### Hand Interaction
```javascript
HAND_REPULSION_RADIUS: 80,      // Distance at which hands affect particles
HAND_FORCE_STRENGTH: 8,         // Strength of the push-away effect
HAND_UPWARD_FORCE: 0.3,         // How much water "flows up" around hands
```

### Visual Style
```javascript
PARTICLE_COLOR: '#4a9eff',      // CSS color for particles
PARTICLE_OPACITY: 0.6,          // Transparency (0-1)
TRAIL_EFFECT: true,             // Motion blur effect
TRAIL_ALPHA: 0.15,              // Lower = longer trails
```

## How It Works

### Hand Tracking
- Uses **MediaPipe Hands** from Google to detect hand landmarks
- Tracks 21 keypoints per hand (finger joints, palm, wrist)
- Runs at ~30-60 FPS depending on your hardware
- Camera feed is processed but never displayed

### Particle Physics
1. Particles spawn at the top of the screen
2. Gravity pulls them downward each frame
3. When a particle gets close to any hand keypoint:
   - Calculate distance to keypoint
   - Apply force pushing particle away (horizontally)
   - Apply slight upward force (creates "flowing" effect)
   - Force is stronger when particle is closer
4. Particles recycle at the bottom and respawn at top

### Force Calculation
For each particle near a hand keypoint:
```
distance = sqrt((particle.x - hand.x)² + (particle.y - hand.y)²)

if distance < REPULSION_RADIUS:
    falloff = (1 - distance/RADIUS)²  // Inverse square falloff
    force = FORCE_STRENGTH × falloff

    // Push away from hand
    particle.vx += (normalized_dx) × force
    particle.vy += (normalized_dy) × force

    // Add upward component
    particle.vy -= UPWARD_FORCE × force
```

## Troubleshooting

**Camera not working:**
- Ensure you clicked "Allow" when prompted for camera access
- Check browser console for errors (F12)
- Try a different browser (Chrome recommended)
- Ensure you're accessing via `localhost` or HTTPS

**Performance issues:**
- Reduce `PARTICLE_COUNT` (try 1000 or 500)
- Set `modelComplexity: 0` in MediaPipe Hands configuration (line ~304)
- Disable `TRAIL_EFFECT` for better performance
- Close other tabs/applications

**Hands not detected:**
- Ensure good lighting
- Keep hands within camera frame
- Try moving hands more slowly
- Adjust `HAND_CONFIDENCE_THRESHOLD` (lower = more sensitive)

**Particles not reacting to hands:**
- Increase `HAND_REPULSION_RADIUS` (try 120)
- Increase `HAND_FORCE_STRENGTH` (try 12)
- Check that hands are being detected (uncomment debug code)

## Debug Mode

To see hand landmarks and repulsion radius, uncomment the debug visualization code at the bottom of the script (lines ~395-417):

```javascript
function drawHandDebug() {
    handsData.forEach(hand => {
        hand.landmarks.forEach((landmark, index) => {
            ctx.fillStyle = index === 9 ? '#ff0000' : '#00ff00';
            ctx.beginPath();
            ctx.arc(landmark.x, landmark.y, 5, 0, Math.PI * 2);
            ctx.fill();

            if (index === 9) {
                ctx.strokeStyle = 'rgba(255, 0, 0, 0.3)';
                ctx.beginPath();
                ctx.arc(landmark.x, landmark.y, CONFIG.HAND_REPULSION_RADIUS, 0, Math.PI * 2);
                ctx.stroke();
            }
        });
    });
}

// Add in animate() function:
drawHandDebug();
```

## Technical Details

- **No external dependencies** - all libraries loaded from CDN
- **MediaPipe Hands** - production-ready ML hand tracking from Google
- **Canvas 2D rendering** - compatible with all modern browsers
- **60 FPS target** - uses requestAnimationFrame for smooth animation
- **Responsive** - automatically adjusts to window size

## License

Free to use and modify for any purpose.

## Credits

- Hand tracking: [MediaPipe Hands](https://google.github.io/mediapipe/solutions/hands.html)
- Created as an interactive art installation

Enjoy creating your digital waterfall! 💧✨
