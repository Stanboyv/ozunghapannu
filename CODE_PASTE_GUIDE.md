# Code Paste Guide - Exact Locations for CRO Implementation

## 📍 Integration Points

This guide shows EXACTLY where each piece of code goes in your project files.

---

## 1️⃣ HTML FILE (`index.html`)

### Location 1: CRO Toggle Button
**Where**: Below Constellation Plot (around line 535)  
**What to find**: `</div>` closing the constellation plot, then immediately:
**Insert this HTML**:

```html
    <!-- CRO Toggle Button -->
    <div style="margin-bottom:16px;display:flex;gap:10px;flex-wrap:wrap;align-items:center;">
      <button id="croBtnToggle" class="btn" style="background:#00ff88;color:#0a0e1a;font-weight:700;">🔌 Open CRO (Oscilloscope)</button>
      <button id="croShowAll" class="btn" style="background:#2f3ea3;padding:8px 12px;font-size:0.9rem;">Show I/Q</button>
      <button id="croShowCarrier" class="btn" style="background:#2f3ea3;padding:8px 12px;font-size:0.9rem;">Show Carrier</button>
    </div>
```

**Before you insert**: Make sure the line after is:
```html
    <!-- CRO Display Container -->
    <div id="croPanel" class="cro-container" style="display:none;">
```

---

### Location 2: CRO Panel Wrapper ID
**Where**: Line where CRO container starts  
**Original**:
```html
    <!-- CRO Display Container -->
    <div class="cro-container">
```

**Change to**:
```html
    <!-- CRO Display Container -->
    <div id="croPanel" class="cro-container" style="display:none;">
```

---

### Location 3: CRO Panel Close Tag
**Where**: Find the closing `</div>` for the CRO container (around line 590)  
**Original**:
```html
        <button id="zoomReset" class="btn" style="background:#2f3ea3;padding:6px 10px;font-size:0.85rem;">↻ Reset</button>
      </div>
    </div>

    <!-- Measurements & Analysis Panel -->
```

**Change to**:
```html
        <button id="zoomReset" class="btn" style="background:#2f3ea3;padding:6px 10px;font-size:0.85rem;">↻ Reset</button>
      </div>
    </div>
    </div><!-- End croPanel -->

    <!-- Measurements & Analysis Panel -->
```

**Note**: Add the extra `</div><!-- End croPanel -->` to close the id="croPanel" wrapper.

---

### Location 4: CROEnhanced Module (JavaScript)
**Where**: In first `<script>` tag, BEFORE `document.addEventListener('DOMContentLoaded'`  
**Insert this code**:

```javascript
/* ====================================================
   ENHANCED CRO (OSCILLOSCOPE) WAVEFORM SYSTEM
   ==================================================== */

// CRO Enhancement Module
const CROEnhanced = {
  isOpen: false,
  showIQ: false,
  showCarrier: false,
  
  /**
   * Toggle CRO panel visibility
   */
  togglePanel() {
    const panel = document.getElementById('croPanel');
    const btn = document.getElementById('croBtnToggle');
    if (!panel || !btn) return;
    
    this.isOpen = !this.isOpen;
    if (this.isOpen) {
      panel.style.display = 'block';
      btn.classList.add('active');
      btn.textContent = '🔌 Close CRO';
      setTimeout(() => {
        const canvas = document.getElementById('waveformCanvas');
        if (canvas) {
          canvas.width = canvas.clientWidth;
          canvas.height = canvas.clientHeight;
        }
      }, 100);
    } else {
      panel.style.display = 'none';
      btn.classList.remove('active');
      btn.textContent = '🔌 Open CRO (Oscilloscope)';
    }
  },
  
  /**
   * Improved waveform rendering with proper scaling and anti-aliasing
   */
  renderWaveform(ctx, waveformData, timePerDiv, voltPerDiv, canvas) {
    if (!ctx || !waveformData || waveformData.length === 0) return;
    
    const width = canvas.width;
    const height = canvas.height;
    const centerX = 0;
    const centerY = height / 2;
    
    // Calculate scale factors
    const fs = 1000; // Sampling frequency in samples per ms (1 KHz)
    const pxPerMs = width / (10 * timePerDiv); // 10 divisions
    const pxPerVolt = height / (10 * voltPerDiv); // 10 divisions
    
    // Smooth the waveform with low-pass filtering to prevent aliasing
    const smoothedData = this.lowPassFilter(waveformData, 0.1);
    
    // Draw waveform trace
    ctx.strokeStyle = '#00ff88';
    ctx.lineWidth = 2.5;
    ctx.lineJoin = 'round';
    ctx.lineCap = 'round';
    ctx.globalAlpha = 0.9;
    
    // Enable some glow effect
    ctx.shadowColor = 'rgba(0, 255, 136, 0.5)';
    ctx.shadowBlur = 6;
    
    ctx.beginPath();
    for (let i = 0; i < smoothedData.length && i < width; i++) {
      const y = smoothedData[i];
      const yPixel = centerY - (y * pxPerVolt);
      
      if (i === 0) {
        ctx.moveTo(i, yPixel);
      } else {
        ctx.lineTo(i, yPixel);
      }
    }
    ctx.stroke();
    
    ctx.globalAlpha = 1.0;
    ctx.shadowBlur = 0;
    ctx.shadowColor = 'transparent';
  },
  
  /**
   * Low-pass filter to smooth waveform and prevent aliasing
   */
  lowPassFilter(data, alpha) {
    if (data.length === 0) return data;
    const filtered = [data[0]];
    
    for (let i = 1; i < data.length; i++) {
      const val = alpha * data[i] + (1 - alpha) * filtered[i - 1];
      filtered.push(val);
    }
    return filtered;
  },
  
  /**
   * Auto-scale for optimal viewing
   */
  autoScale(data, targetHeight) {
    if (!data || data.length === 0) return { scale: 1, offset: 0 };
    
    const max = Math.max(...data.map(d => Math.abs(d)));
    if (max === 0) return { scale: 1, offset: 0 };
    
    const scale = (targetHeight / 2) / max;
    return { scale, offset: 0 };
  }
};

```

**Insert before this line**:
```javascript
/* ---- Utility & Simulation Engine ---- */
document.addEventListener('DOMContentLoaded', () => {
```

---

### Location 5: CRO Control Element References
**Where**: Inside DOMContentLoaded, find where controls are declared (around line 1050)

**Original**: 
```javascript
  const zoomOut = document.getElementById('zoomOut');

  // Outputs
```

**Change to**:
```javascript
  const zoomOut = document.getElementById('zoomOut');

  // CRO Controls
  const croBtnToggle = document.getElementById('croBtnToggle');
  const croShowAll = document.getElementById('croShowAll');
  const croShowCarrier = document.getElementById('croShowCarrier');
  const croPanel = document.getElementById('croPanel');

  // Outputs
```

---

### Location 6: CRO Button Event Listeners
**Where**: After toggleNoisy event listener (around line 2025)

**Find this**:
```javascript
  // toggle noisy: replot waveform to show/hide
  toggleNoisy.addEventListener('change', ()=> {
    // ... code ...
  });

  // initial UI values
```

**Insert before "// initial UI values"**:
```javascript

  // ===== CRO BUTTON HANDLERS =====
  if (croBtnToggle) {
    croBtnToggle.addEventListener('click', () => {
      CROEnhanced.togglePanel();
    });
  }

  if (croShowAll) {
    croShowAll.addEventListener('click', () => {
      CROEnhanced.showIQ = !CROEnhanced.showIQ;
      croShowAll.style.background = CROEnhanced.showIQ ? '#00ff88' : '#2f3ea3';
      croShowAll.style.color = CROEnhanced.showIQ ? '#0a0e1a' : '#fff';
    });
  }

  if (croShowCarrier) {
    croShowCarrier.addEventListener('click', () => {
      CROEnhanced.showCarrier = !CROEnhanced.showCarrier;
      croShowCarrier.style.background = CROEnhanced.showCarrier ? '#00eaff' : '#2f3ea3';
      croShowCarrier.style.color = CROEnhanced.showCarrier ? '#0a0e1a' : '#fff';
    });
  }

```

---

### Location 7: Improved drawOscilloscope Function
**Where**: Find `function drawOscilloscope(timestamp)` (around line 1301)

**Replace the entire function with**:
```javascript
  function drawOscilloscope(timestamp) {
    if (!wfCtx) return;
    // check for resize if container size changed
    if (waveformCanvas.width !== waveformCanvas.clientWidth || waveformCanvas.height !== waveformCanvas.clientHeight) {
      resizeWaveformCanvas();
    }
    const dt = timestamp - lastFrameTime;
    lastFrameTime = timestamp;

    if (waveformRunning) {
      // update sweep position
      const timeDiv = SimulationEngine.timePerDiv; // ms per division
      const divisions = 10; // minor divisions across width
      const pxPerMs = waveformCanvas.width / (divisions * timeDiv);
      sweepOffset = (sweepOffset + dt * pxPerMs) % waveformCanvas.width;

      // clear with slight persistence (CRT effect)
      wfCtx.fillStyle = 'rgba(5,7,19,0.15)';
      wfCtx.fillRect(0, 0, waveformCanvas.width, waveformCanvas.height);

      // Improved scaling: prevent clipping with automatic headroom
      const heightHalf = waveformCanvas.height / 2;
      let voltScale = SimulationEngine.voltPerDiv;
      
      // Auto-adjust if amplitude is too large (prevent clipping)
      const amp_ch1 = Number(ch1Amp.value) || 1;
      const maxSignal = amp_ch1 * 1.2; // Add 20% headroom
      if (maxSignal > voltScale * 5) { // If signal would take > half screen
        voltScale = maxSignal / 5;
        SimulationEngine.voltPerDiv = voltScale;
      }
      
      const pxPerVolt = waveformCanvas.height / (divisions * voltScale);
      const clippingThreshold = heightHalf * 0.95; // Leave 5% margin
      
      // common line styling + glow with improved quality
      wfCtx.lineWidth = 2.2;
      wfCtx.lineJoin = 'round';
      wfCtx.lineCap = 'round';
      wfCtx.imageSmoothingEnabled = true;
      wfCtx.imageSmoothingQuality = 'high';

      if (showCH1) {
        wfCtx.strokeStyle = '#00ff88'; // neon green
        wfCtx.shadowColor = 'rgba(0,255,136,0.7)';
        wfCtx.shadowBlur = 10;
        wfCtx.globalAlpha = 0.95;
        wfCtx.beginPath();
        
        let isFirstPoint = true;
        for (let x = 0; x < waveformCanvas.width; x += 1) {
          const t = ((x + sweepOffset) / pxPerMs) / 1000; // seconds
          const fc_ch1 = Number(ch1Freq.value) || 1000; // Hz (user-controlled)
          const phase_ch1 = (Number(ch1Phase.value) || 0) * Math.PI / 180; // convert to radians
          
          // Generate clean sinusoid: y = A * sin(2πf*t + φ)
          let y1 = amp_ch1 * Math.sin(2 * Math.PI * fc_ch1 * t + phase_ch1);
          
          // Clamp to prevent distortion at display edges
          y1 = Math.max(-5, Math.min(5, y1)); // Clamp to reasonable range
          
          const yPx = heightHalf - (y1 * pxPerVolt);
          
          if (isFirstPoint) {
            wfCtx.moveTo(x, yPx);
            isFirstPoint = false;
          } else {
            wfCtx.lineTo(x, yPx);
          }
        }
        wfCtx.stroke();
        wfCtx.globalAlpha = 1.0;
      }
      
      if (showCH2 && lastCleanWave) {
        wfCtx.strokeStyle = '#00eaff'; // cyan/blue
        wfCtx.shadowColor = 'rgba(0,234,255,0.7)';
        wfCtx.shadowBlur = 10;
        wfCtx.globalAlpha = 0.85;
        wfCtx.beginPath();
        
        const fs = lastCleanWave.fs || 1000;
        const ch2_offset = Number(ch2Offset?.value) || 0;
        const ch2_scale = Number(ch2Scale?.value) || 1;
        
        let isFirstPoint = true;
        for (let x = 0; x < waveformCanvas.width; x += 1) {
          const t = ((x + sweepOffset) / pxPerMs) / 1000;
          const idx = Math.floor(t * fs) % Math.max(1, lastCleanWave.y.length);
          
          // scale and offset CH2 signal with clipping prevention
          let y2 = ((lastCleanWave.y[idx] || 0) * ch2_scale + ch2_offset);
          y2 = Math.max(-5, Math.min(5, y2)); // Clamp to prevent distortion
          
          const y2Px = heightHalf - (y2 * pxPerVolt);
          
          if (isFirstPoint) {
            wfCtx.moveTo(x, y2Px);
            isFirstPoint = false;
          } else {
            wfCtx.lineTo(x, y2Px);
          }
        }
        wfCtx.stroke();
        wfCtx.globalAlpha = 1.0;
      }
      
      // reset shadow and effects
      wfCtx.shadowBlur = 0;
      wfCtx.shadowColor = 'transparent';
      wfCtx.globalAlpha = 1.0;
    }
    requestAnimationFrame(drawOscilloscope);
  }
```

---

### Location 8: Improved updateCRODisplayIndicators Function
**Where**: Find `function updateCRODisplayIndicators()` (around line 1448)

**Replace the entire function with**:
```javascript
  function updateCRODisplayIndicators() {
    const symbolRateVal = Number(symbolRate.value) || 1000;
    const snrVal = Number(snrEl.value) || 20;
    const ampValNum = (amplitude ? Number(amplitude.value) : 1) || 1;
    
    // ===== IMPROVED TIME/DIV CALCULATION =====
    // Time/Div based on symbol rate for proper synchronization
    const timePerSymbolMs = 1000 / symbolRateVal;
    let timePerDiv = Math.max(0.05, timePerSymbolMs / 2); // At least 2 symbols per screen
    
    // Apply zoom scaling
    if (zoomScale && zoomScale > 0) {
      timePerDiv = timePerDiv / zoomScale; // Zoom in = finer resolution
    }
    
    // Format time value
    const timePerDivStr = timePerDiv < 0.1 
      ? (timePerDiv * 1000).toFixed(2) + ' µs' 
      : timePerDiv.toFixed(2) + ' ms';
    
    document.getElementById('timePerDiv').textContent = timePerDivStr;
    SimulationEngine.timePerDiv = Math.max(0.01, timePerDiv); // Store actual value (minimum 0.01 ms)
    
    // ===== IMPROVED VOLT/DIV CALCULATION =====
    // Critical: Prevent clipping by calculating based on signal amplitude
    const signalPeak = ampValNum * 1.5; // Expected peak (accounting for modulation)
    const noiseAmplitude = Math.sqrt(1 / (2 * Math.pow(10, snrVal / 10))); // Noise std dev
    const totalRangeNeeded = signalPeak + (noiseAmplitude * 3); // 3-sigma headroom for noise
    
    // Calculate voltage per division to fit signal with 20% margin
    let voltPerDiv = totalRangeNeeded / 8; // Leave room for peaks
    voltPerDiv = Math.max(0.1, Math.min(5, voltPerDiv)); // Clamp to reasonable range
    
    // Apply zoom scaling for voltage
    if (zoomScale && zoomScale > 0) {
      voltPerDiv = voltPerDiv / zoomScale; // Zoom in = finer resolution
    }
    
    // Ensure minimum for display
    voltPerDiv = Math.max(0.05, voltPerDiv);
    
    document.getElementById('voltPerDiv').textContent = voltPerDiv.toFixed(3) + ' V';
    SimulationEngine.voltPerDiv = voltPerDiv;
  }
```

---

## 2️⃣ CSS FILE (`qam.css`)

### Location 1: Add New CSS Classes
**Where**: At the END of the file (after line ~2500)

**Add this CSS**:
```css
/* ============================================
   IMPROVED CRO WAVEFORM RENDERING
   ============================================ */

/* Enhanced canvas anti-aliasing and smoothing */
.cro-waveform-canvas {
    image-rendering: high-quality;
    image-rendering: crisp-edges;
    filter: drop-shadow(0 0 8px rgba(0, 255, 136, 0.2));
}

/* Improved CRO display wrapper */
.cro-display-wrapper {
    background: #050713;
    position: relative;
    width: 100%;
    height: 400px;
    border: 1px solid rgba(0, 255, 247, 0.2);
    border-radius: 8px;
    overflow: hidden;
    box-shadow: inset 0 0 15px rgba(0, 255, 247, 0.05);
}

/* CRO Grid improvements */
.cro-grid-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 1;
    background: 
        repeating-linear-gradient(
            0deg,
            transparent,
            transparent 19px,
            rgba(0, 255, 136, 0.08) 19px,
            rgba(0, 255, 136, 0.08) 20px
        ),
        repeating-linear-gradient(
            90deg,
            transparent,
            transparent 19px,
            rgba(0, 255, 136, 0.08) 19px,
            rgba(0, 255, 136, 0.08) 20px
        ),
        repeating-linear-gradient(
            0deg,
            transparent,
            transparent 99px,
            rgba(0, 255, 136, 0.15) 99px,
            rgba(0, 255, 136, 0.15) 100px
        ),
        repeating-linear-gradient(
            90deg,
            transparent,
            transparent 99px,
            rgba(0, 255, 136, 0.15) 99px,
            rgba(0, 255, 136, 0.15) 100px
        );
}

/* CRO Toggle Button Style */
#croBtnToggle {
    transition: all 0.3s ease;
    box-shadow: 0 0 12px rgba(0, 255, 136, 0.4);
}

#croBtnToggle:hover {
    background: #00dd77 !important;
    box-shadow: 0 0 20px rgba(0, 255, 136, 0.8);
    transform: scale(1.05);
}

#croBtnToggle.active {
    background: #ff6b6b !important;
    color: #fff !important;
}

/* Show/Hide Channel Buttons */
#croShowAll, #croShowCarrier {
    transition: all 0.2s ease;
}

#croShowAll:hover, #croShowCarrier:hover {
    background: #667eea !important;
    box-shadow: 0 0 12px rgba(102, 126, 234, 0.5);
}

/* I/Q Component visualization container */
.iq-components-display {
    background: #0a0e1a;
    border: 1px solid #00eaff;
    border-radius: 8px;
    padding: 12px;
    margin-top: 12px;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 16px;
}

.iq-component-box {
    background: rgba(0, 255, 247, 0.05);
    border: 1px solid rgba(0, 255, 247, 0.2);
    border-radius: 6px;
    padding: 10px;
    font-size: 0.85rem;
}

.iq-component-box h4 {
    color: #00eaff;
    margin: 0 0 6px 0;
    font-size: 0.95rem;
}

.iq-component-canvas {
    width: 100%;
    height: 150px;
    background: #050713;
    border: 1px solid rgba(0, 255, 247, 0.1);
    border-radius: 4px;
    margin-top: 6px;
}

/* Waveform info panel */
.waveform-info-panel {
    background: rgba(0, 255, 247, 0.05);
    border: 1px solid rgba(0, 255, 247, 0.1);
    border-radius: 6px;
    padding: 12px;
    margin-top: 12px;
    font-size: 0.85rem;
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: 12px;
}

.waveform-stat {
    color: #e5e7eb;
}

.waveform-stat-label {
    color: #00ff88;
    font-weight: 600;
}

.waveform-stat-value {
    color: #00eaff;
    font-family: monospace;
    font-weight: 700;
}

/* Smooth transitions for panel visibility */
#croPanel {
    transition: all 0.3s ease-out;
    opacity: 1;
}

#croPanel.hidden {
    opacity: 0;
    pointer-events: none;
    max-height: 0;
    overflow: hidden;
}
```

---

## ✅ Verification

After pasting all code:

1. **Open** `index.html` in browser
2. **Go to** "Simulation" tab
3. **Look for**:
   - ✅ Green "🔌 Open CRO (Oscilloscope)" button below constellation
   - ✅ Blue "Show I/Q" button
   - ✅ Blue "Show Carrier" button
4. **Test**:
   - Click "▶ Run Simulation"
   - Click green CRO button
   - CRO panel should appear
   - Waveform should render
5. **Check console**: F12 → Console tab → Should be NO errors

---

## 🎯 Summary of Edits

| File | Location | Lines | Change Type |
|------|----------|-------|-------------|
| index.html | Line 536 | 8 | INSERT |
| index.html | Line 545 | 2 | MODIFY |
| index.html | Line 592 | 1 | INSERT |
| index.html | Line 896 | 85 | INSERT |
| index.html | Line 1055 | 5 | INSERT |
| index.html | Line 2025 | 24 | INSERT |
| index.html | Line 1301 | 80 | REPLACE |
| index.html | Line 1448 | 50 | REPLACE |
| qam.css | End | 120 | INSERT |

**Total Changes**: 9 edits  
**Total New Lines**: ~375  
**Estimated Time**: 15-20 minutes  
**Difficulty**: Medium  
**Risk Level**: Low (all additions, no removals)

---

**Ready to implement? Start with HTML Location 1! 🚀**
