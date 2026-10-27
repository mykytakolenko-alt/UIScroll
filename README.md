# UIScroll

Make your website impossible to leave.

**UIScroll** is a lightweight web component designed to hook users into scrolling your site. By combining smooth visual animations with simulated physical haptic feedback, it turns a standard webpage scroll into a highly satisfying, tactile, and addictive experience.

Watch demo:


https://github.com/user-attachments/assets/bb5b0090-eff3-4b89-93ec-8de73e091361




Try it live here (better on your smartphone): https://zesty-churros-cb9de5.netlify.app

## ✨ Why It Works

- **Physical Feedback**: Triggers real device vibration (`navigator.vibrate`) on mobile as you scroll through content zones.
- **Audio Illusion**: Generates crisp, mechanical "ticks" using the Web Audio API (no external audio files needed).
- **Visual Satisfaction**: Smooth gradient progress bars and pulsing animations instantly reward the user's movement.
- **Zero Dependencies**: Pure HTML, CSS, and Vanilla JavaScript. No frameworks, no build steps.

## 🚀 How to Use

1. Just copy and paste the JS code below into your project.
   ```js
   const startScreen = document.getElementById('start');
    const startBtn = document.getElementById('startBtn');
    const progress = document.getElementById('progress');
    const value = document.getElementById('value');
    const scroll = document.getElementById('scroll');
    const status = document.getElementById('status');
    const canvas = document.getElementById('confetti');
    const ctx = canvas.getContext('2d');
    
    let audioCtx = null, lastScroll = 0, lastTick = 0, currentPct = 0, targetPct = 0;
    const maxScroll = () => scroll.scrollHeight - innerHeight;
    
    window.addEventListener('resize', () => { canvas.width = innerWidth; canvas.height = innerHeight; });
    canvas.width = innerWidth; canvas.height = innerHeight;

    // Confetti Engine
    const particles = [], colors = ['#0a84ff', '#5e5ce6', '#bf5af2', '#ff375f', '#ff9f00', '#30d158'];
    function createConfetti() {
      particles.length = 0;
      for (let i = 0; i < 150; i++) {
        const angle = Math.random() * Math.PI * 2, speed = 4 + Math.random() * 8;
        particles.push({
          x: innerWidth / 2, y: innerHeight / 2,
          vx: Math.cos(angle) * speed, vy: Math.sin(angle) * speed - 3,
          size: 6 + Math.random() * 8, color: colors[Math.floor(Math.random() * colors.length)],
          rotation: Math.random() * 360, rotationSpeed: (Math.random() - 0.5) * 15,
          life: 1, decay: 0.008 + Math.random() * 0.008
        });
      }
      animateConfetti();
    }
    function animateConfetti() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      let alive = false;
      for (const p of particles) {
        if (p.life <= 0) continue;
        alive = true;
        p.x += p.vx; p.y += p.vy; p.vy += 0.25; p.vx *= 0.98;
        p.rotation += p.rotationSpeed; p.life -= p.decay;
        ctx.save(); ctx.translate(p.x, p.y); ctx.rotate((p.rotation * Math.PI) / 180);
        ctx.globalAlpha = p.life; ctx.fillStyle = p.color;
        ctx.fillRect(-p.size / 2, -p.size / 2, p.size, p.size * 0.6);
        ctx.restore();
      }
      if (alive) requestAnimationFrame(animateConfetti);
      else ctx.clearRect(0, 0, canvas.width, canvas.height);
    }

    // Audio & Taptic Engine
    function initAudio() {
      if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      if (audioCtx.state === 'suspended') audioCtx.resume();
      const buffer = audioCtx.createBuffer(1, 1, 22050), source = audioCtx.createBufferSource();
      source.buffer = buffer; source.connect(audioCtx.destination); source.start(0);
    }

    function playTaptic(intensity = 1) {
      if (!audioCtx) return;
      if (audioCtx.state === 'suspended') audioCtx.resume();
      
      const now = audioCtx.currentTime;
      const filter = audioCtx.createBiquadFilter(); filter.type = 'lowpass'; filter.frequency.value = 1500;
      const osc = audioCtx.createOscillator(); const gain = audioCtx.createGain();
      
      osc.type = 'sine';
      osc.frequency.setValueAtTime(140 * intensity, now);
      osc.frequency.exponentialRampToValueAtTime(40 * intensity, now + 0.12);
      
      gain.gain.setValueAtTime(0, now);
      gain.gain.linearRampToValueAtTime(0.9 * intensity, now + 0.01);
      gain.gain.exponentialRampToValueAtTime(0.001, now + 0.15);
      
      osc.connect(gain).connect(filter).connect(audioCtx.destination);
      osc.start(now); osc.stop(now + 0.15);
      
      if (navigator.vibrate) { try { navigator.vibrate(20 * intensity); } catch (e) {} }
    }

    // Main Loops & Handlers
    function handleStart(e) {
      e.preventDefault();
      initAudio();
      playTaptic(1.2);
      createConfetti();
      startScreen.classList.add('hidden');
      
      function animate() {
        currentPct += (targetPct - currentPct) * 0.15;
        progress.style.width = `${currentPct}%`;
        value.textContent = Math.round(currentPct);
        requestAnimationFrame(animate);
      }
      requestAnimationFrame(animate);
      
      setTimeout(() => {
        status.classList.add('show');
        setTimeout(() => status.classList.remove('show'), 3000);
      }, 400);
    }

    startBtn.addEventListener('click', handleStart);
    startBtn.addEventListener('touchstart', handleStart, { passive: false });

    scroll.addEventListener('scroll', () => {
      const s = scroll.scrollTop, total = maxScroll();
      if (total <= 0) return;
      targetPct = (s / total) * 100;
      
      const now = Date.now(), zone = Math.floor(s / 50), lastZone = Math.floor(lastScroll / 50);
      if (zone !== lastZone && now - lastTick > 70) {
        playTaptic(1);
        value.classList.add('pulse');
        setTimeout(() => value.classList.remove('pulse'), 250);
        lastTick = now;
      }
      
      if (Math.floor(targetPct / 10) !== Math.floor((lastScroll / total * 100) / 10) && targetPct > 0) {
        playTaptic(1.6);
      }
      lastScroll = s;
    }, { passive: true });
2. Сopy and paste the CSS code below into your project.
    ```CSS
   * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
    body { height: 100vh; overflow: hidden; background: #000; font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', sans-serif; color: #fff; }
    .confetti { position: fixed; inset: 0; z-index: 2000; pointer-events: none; }
    
    .start {
      position: fixed; inset: 0; background: #000; display: flex; flex-direction: column;
      align-items: center; justify-content: center; z-index: 1500; padding: 40px; text-align: center;
      transition: opacity 0.6s cubic-bezier(0.4, 0, 0.2, 1);
    }
    .start.hidden { opacity: 0; pointer-events: none; }
    .start h1 {
      font-size: 52px; font-weight: 200; margin-bottom: 16px;
      background: linear-gradient(135deg, #0a84ff, #bf5af2);
      -webkit-background-clip: text; background-clip: text; -webkit-text-fill-color: transparent;
    }
    .start p { font-size: 17px; opacity: 0.6; margin-bottom: 48px; line-height: 1.5; max-width: 320px; }
    .start-btn {
      background: linear-gradient(135deg, #0a84ff, #5e5ce6); color: #fff; border: none;
      padding: 20px 60px; font-size: 18px; font-weight: 600; border-radius: 30px; cursor: pointer;
      box-shadow: 0 10px 40px rgba(10, 132, 255, 0.4); transition: transform 0.2s cubic-bezier(0.34, 1.56, 0.64, 1);
    }
    .start-btn:active { transform: scale(0.95); }
    
    .progress {
      position: fixed; top: 0; left: 0; height: 3px; width: 0%; z-index: 100;
      background: linear-gradient(90deg, #0a84ff, #5e5ce6, #bf5af2);
      box-shadow: 0 0 15px rgba(10, 132, 255, 0.6);
    }
    .value {
      position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%) scale(1);
      font-size: 200px; font-weight: 100; color: rgba(255, 255, 255, 0.12); pointer-events: none;
      transition: transform 0.4s cubic-bezier(0.34, 1.56, 0.64, 1), color 0.4s ease;
    }
    .value.pulse {
      transform: translate(-50%, -50%) scale(1.12);
      color: rgba(10, 132, 255, 0.5); text-shadow: 0 0 60px rgba(10, 132, 255, 0.4);
    }
    
    .scroll { height: 100vh; overflow-y: auto; -webkit-overflow-scrolling: touch; overscroll-behavior: contain; }
    .content { height: 6000px; }
    
    .status {
      position: fixed; bottom: 30px; left: 50%; transform: translateX(-50%) translateY(20px);
      background: rgba(48, 209, 88, 0.15); color: #30d158; padding: 10px 20px; border-radius: 20px;
      font-size: 14px; font-weight: 600; opacity: 0; transition: opacity 0.4s ease, transform 0.4s ease;
      pointer-events: none; backdrop-filter: blur(10px); z-index: 50;
    }
    .status.show { opacity: 1; transform: translateX(-50%) translateY(0); }
   

## 💡 The Goal

To increase user engagement and time-on-site by making the simple act of scrolling feel premium, responsive, and genuinely fun.

## 👤 Author

**Mykyta Kolenko**

## 📜 License

Distributed under the **Apache License 2.0**.  
See the [LICENSE](LICENSE) file for details.
