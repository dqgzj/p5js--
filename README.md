<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>光涟</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/p5.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400&display=swap" rel="stylesheet">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: #000;
            color: #fff;
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            overflow: hidden;
            cursor: default;
        }
        .title {
            position: absolute;
            top: 20px;
            left: 20px;
            font-size: 1.8rem;
            color: #fff;
            font-weight: 300;
            letter-spacing: 3px;
            z-index: 10;
        }
        #canvas-container {
            position: relative;
            width: 600px;
            height: 800px;
            overflow: hidden;
            background: #000;
        }
    </style>
</head>
<body>
    <div class="title">光涟</div>
    <div id="canvas-container"></div>

    <script>
        let particles = [];
        const maxParticles = 2500;
        let ripples = [];
        let trails = [];
        let isMousePressed = false;
        let lastTrailTime = 0;
        
        function setup() {
            const canvas = createCanvas(600, 800);
            canvas.parent('canvas-container');
            background(0);
            
            textAlign(CENTER, CENTER);
            textFont('Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif');
        }
        
        function draw() {
            // 纯黑背景，不留痕迹
            clear();
            background(0);
            
            updateTrails();
            updateRipples();
            updateParticles();
            
            if (isMousePressed && mouseIsPressed && millis() - lastTrailTime > 50) {
                createTrail(mouseX, mouseY);
                lastTrailTime = millis();
            }
        }
        
        function updateTrails() {
            for (let i = trails.length - 1; i >= 0; i--) {
                let trail = trails[i];
                trail.life -= 3;
                
                if (trail.life > 120 && frameCount % 3 === 0) {
                    createTrailParticles(trail);
                }
                
                if (trail.life <= 0) {
                    trails.splice(i, 1);
                }
            }
        }
        
        function easeOutQuad(t) {
            return t * (2 - t);
        }
        
        function easeInOutCubic(t) {
            return t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2;
        }
        
        function createTrail(x, y) {
            trails.push({
                x: x,
                y: y,
                life: 180,
                width: random(10, 18)
            });
        }
        
        function createTrailParticles(trail) {
            for (let side = 0; side < 2; side++) {
                const angleOffset = side === 0 ? -PI/2 : PI/2;
                
                for (let i = 0; i < 2; i++) {
                    const angle = random(PI) + angleOffset;
                    const distance = random(trail.width * 0.3, trail.width * 0.6);
                    const x = trail.x + cos(angle) * distance;
                    const y = trail.y + sin(angle) * distance;
                    
                    const distanceFromCenter = dist(x, y, trail.x, trail.y);
                    const maxDistance = trail.width * 0.6;
                    const radialFactor = distanceFromCenter / maxDistance;
                    const lifeMultiplier = 1 - radialFactor * 0.7;
                    
                    particles.push({
                        x: x,
                        y: y,
                        char: random() > 0.5 ? '开' : '关',
                        size: random(4, 7),
                        baseOpacity: 180,
                        vx: cos(angle) * random(0.6, 1.2) + random(-0.15, 0.15),
                        vy: sin(angle) * random(0.6, 1.2) + random(-0.15, 0.15),
                        maxLife: trail.life * lifeMultiplier,
                        life: trail.life * lifeMultiplier,
                        rotation: random(-0.02, 0.02),
                        spinSpeed: random(-0.008, 0.008),
                        radialFactor: radialFactor,
                        fadeStartTime: 0.3 + radialFactor * 0.4,
                        fadeDuration: 0.4 - radialFactor * 0.2
                    });
                }
            }
        }
        
        function updateRipples() {
            for (let i = ripples.length - 1; i >= 0; i--) {
                let ripple = ripples[i];
                const progress = ripple.radius / ripple.maxRadius;
                ripple.radius += ripple.speed * (1 + easeOutQuad(progress) * 0.4);
                ripple.life = 255 * (1 - easeInOutCubic(progress));
                
                if (ripple.life <= 0) {
                    ripples.splice(i, 1);
                    continue;
                }
                
                if (ripple.radius % 4 < ripple.speed && particles.length < maxParticles) {
                    createRippleParticles(ripple);
                }
                
                drawRippleLayers(ripple);
            }
        }
        
        function drawRippleLayers(ripple) {
            // 核心光斑效果
            if (ripple.radius < 25) {
                const coreProgress = ripple.radius / 25;
                const coreAlpha = 80 * (1 - easeOutQuad(coreProgress));
                
                // 多层光晕效果
                for (let i = 0; i < 3; i++) {
                    const glowSize = ripple.radius * (1 + i * 0.3);
                    const glowAlpha = coreAlpha * (0.4 - i * 0.1);
                    noStroke();
                    fill(255, glowAlpha * 0.1);
                    ellipse(ripple.x, ripple.y, glowSize);
                }
            }
            
            // 涟漪轮廓
            noFill();
            for (let i = 0; i < 2; i++) {
                const offset = i * 8;
                const alpha = ripple.life * (0.08 - i * 0.03);
                stroke(255, alpha);
                strokeWeight(0.5);
                ellipse(ripple.x, ripple.y, (ripple.radius - offset) * 2);
            }
        }
        
        function createRippleParticles(ripple) {
            const progress = ripple.radius / ripple.maxRadius;
            const density = 15 * (1 - easeOutQuad(progress));
            const points = max(1, floor(density));
            
            for (let i = 0; i < points; i++) {
                const angle = random(TWO_PI);
                const distance = ripple.radius + random(-4, 4);
                const x = ripple.x + cos(angle) * distance;
                const y = ripple.y + sin(angle) * distance;
                
                const distanceFromCenter = dist(x, y, ripple.x, ripple.y);
                const maxDistance = ripple.maxRadius;
                const radialFactor = distanceFromCenter / maxDistance;
                const lifeMultiplier = 1 - radialFactor * 0.6;
                
                const centerDensity = 0.9 * (1 - easeOutQuad(radialFactor * 1.5));
                if (random() > centerDensity) continue;
                
                particles.push({
                    x: x,
                    y: y,
                    char: random() > 0.5 ? '开' : '关',
                    size: random(4, 7),
                    baseOpacity: 200,
                    vx: cos(angle) * random(1.0, 2.0) + random(-0.2, 0.2),
                    vy: sin(angle) * random(1.0, 2.0) + random(-0.2, 0.2),
                    maxLife: ripple.life * lifeMultiplier,
                    life: ripple.life * lifeMultiplier,
                    rotation: random(-0.02, 0.02),
                    spinSpeed: random(-0.01, 0.01),
                    radialFactor: radialFactor,
                    fadeStartTime: 0.2 + radialFactor * 0.5,
                    fadeDuration: 0.3 + radialFactor * 0.2
                });
            }
        }
        
        function updateParticles() {
            particles.sort((a, b) => b.radialFactor - a.radialFactor);
            
            for (let i = particles.length - 1; i >= 0; i--) {
                let p = particles[i];
                
                p.x += p.vx;
                p.y += p.vy;
                p.rotation += p.spinSpeed;
                p.vx *= 0.97;
                p.vy *= 0.97;
                
                const lifeProgress = 1 - (p.life / p.maxLife);
                
                let fadeSpeed = 1.0;
                if (p.radialFactor > 0.7) {
                    fadeSpeed = 2.2;
                } else if (p.radialFactor > 0.4) {
                    fadeSpeed = 1.5;
                }
                
                if (lifeProgress > p.fadeStartTime) {
                    const fadeProgress = (lifeProgress - p.fadeStartTime) / p.fadeDuration;
                    fadeSpeed *= (1 + easeOutQuad(fadeProgress) * 1.8);
                }
                
                p.life -= fadeSpeed;
                
                let opacity = p.baseOpacity;
                if (lifeProgress > p.fadeStartTime) {
                    const fadeProgress = (lifeProgress - p.fadeStartTime) / p.fadeDuration;
                    opacity *= (1 - easeInOutCubic(fadeProgress));
                }
                
                if (p.life > 0) {
                    push();
                    translate(p.x, p.y);
                    rotate(p.rotation);
                    
                    const brightness = 255 * (0.95 + 0.05 * (p.life / p.maxLife));
                    fill(brightness, opacity);
                    noStroke();
                    
                    // 根据生命值微调大小
                    const scale = 0.85 + 0.15 * (p.life / p.maxLife);
                    textSize(p.size * scale);
                    
                    text(p.char, 0, 0);
                    pop();
                } else {
                    particles.splice(i, 1);
                }
            }
        }
        
        function mousePressed() {
            if (mouseButton === LEFT) {
                isMousePressed = true;
                createClickRipple(mouseX, mouseY);
            }
            return false;
        }
        
        function mouseReleased() {
            isMousePressed = false;
            return false;
        }
        
        function createClickRipple(x, y) {
            // 主要光斑
            ripples.push({
                x: x,
                y: y,
                radius: 0,
                startRadius: 0,
                maxRadius: random(100, 150),
                speed: random(2.5, 4),
                life: 255
            });
            
            // 次级光斑
            for (let i = 0; i < 2; i++) {
                setTimeout(() => {
                    ripples.push({
                        x: x + random(-8, 8),
                        y: y + random(-8, 8),
                        radius: 0,
                        startRadius: 0,
                        maxRadius: random(30, 60),
                        speed: random(1.5, 2.5),
                        life: 180
                    });
                }, i * 60);
            }
            
            createInitialParticles(x, y);
        }
        
        function createInitialParticles(x, y) {
            const initialParticles = 60;
            for (let i = 0; i < initialParticles; i++) {
                const angle = random(TWO_PI);
                const distance = random(12);
                const px = x + cos(angle) * distance;
                const py = y + sin(angle) * distance;
                
                const radialFactor = distance / 12;
                
                particles.push({
                    x: px,
                    y: py,
                    char: random() > 0.5 ? '开' : '关',
                    size: random(5, 8),
                    baseOpacity: 240,
                    vx: cos(angle) * random(1.2, 2.5),
                    vy: sin(angle) * random(1.2, 2.5),
                    maxLife: random(150, 220) * (1 - radialFactor * 0.5),
                    life: random(150, 220) * (1 - radialFactor * 0.5),
                    rotation: random(TWO_PI),
                    spinSpeed: random(-0.015, 0.015),
                    radialFactor: radialFactor,
                    fadeStartTime: 0.2 + radialFactor * 0.6,
                    fadeDuration: 0.3 + radialFactor * 0.2
                });
            }
        }
        
        function mouseDragged() {
            if (mouseButton === LEFT) {
                if (frameCount % 5 === 0) {
                    createTrail(mouseX, mouseY);
                }
            }
            return false;
        }
    </script>
</body>
</html>
