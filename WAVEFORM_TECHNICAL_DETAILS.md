# QAM Waveform Implementation Details

## 1. PROBLEM ANALYSIS

### Old Implementation Issues
```
generateWaveform(IQSymbols, fc, symbolDurMs, samplesPerSymbol=64)

Sampling frequency: fs = 64 / Ts (symbol)

Example calculation (Problem):
  fc = 2000 Hz (carrier)
  Symbol Rate = 2000 sym/s
  Ts = 0.0005 s (symbol duration)
  
  fs_old = 64 / 0.0005 = 128,000 Hz  ← Seems OK
  
  BUT if fc = 5000 Hz:
  Nyquist requirement: fs ≥ 2×5000 = 10,000 Hz
  fs_old = 128,000 Hz  ← Still OK, but arbitrary
  
  BUT the fixed samplesPerSymbol=64 means:
  For lower symbol rates: fs could be too LOW
  
  Example with 500 sym/s:
  Ts = 0.002 s
  fs_old = 64 / 0.002 = 32,000 Hz  ← Fine
  
  BUT with 10 sym/s (very slow):
  Ts = 0.1 s
  fs_old = 64 / 0.1 = 640 Hz
  For fc=5000 Hz: Nyquist needs 10,000 Hz  ← ALIASING!
```

### New Implementation Benefits
```
generateWaveform(IQSymbols, fc, symbolDurMs, samplesPerSymbol=null)

Sampling frequency: fs = max(10×fc, 64/Ts), rounded up to power of 2

Example calculation (Solution):
  fc = 2000 Hz
  Symbol Rate = 2000 sym/s
  Ts = 0.0005 s
  
  fs_min = max(10×2000, 64/0.0005)
         = max(20,000, 128,000)
         = 128,000 Hz
  
  fs = 2^ceil(log2(128000)) = 2^17 = 131,072 Hz  ✓ Good
  
  Even with fc=5000 Hz, Ts=0.1 s (slow modulation):
  fs_min = max(10×5000, 64/0.1)
         = max(50,000, 640)
         = 50,000 Hz
  
  fs = 2^ceil(log2(50000)) = 2^16 = 65,536 Hz  ✓ Prevents aliasing
```

---

## 2. WAVEFORM GENERATION ALGORITHM

### Pseudocode
```
function generateWaveform(IQSymbols, fc, symbolDurMs):
    Ts = symbolDurMs / 1000  // Convert to seconds
    
    // Calculate sampling frequency with Nyquist margin
    fs_min = max(10 * fc, 64 / Ts)
    fs = 2^ceil(log2(fs_min))  // Round up to power of 2
    
    sps = round(fs * Ts)  // Samples per symbol
    dt = 1 / fs  // Time step
    
    t = []  // Time array
    y = []  // Modulated waveform
    I_env = []  // I-component envelope
    Q_env = []  // Q-component envelope
    
    time = 0
    for each symbol k in IQSymbols:
        I = IQSymbols[k].I
        Q = IQSymbols[k].Q
        
        for sample n from 0 to sps-1:
            // Store time and component values
            t.push(time)
            I_env.push(I)
            Q_env.push(Q)
            
            // Apply QAM modulation equation:
            // s(t) = I(t)×cos(2πf_c×t) - Q(t)×sin(2πf_c×t)
            
            carrier_I = I * cos(2π*fc*time)
            carrier_Q = -Q * sin(2π*fc*time)
            s_t = carrier_I + carrier_Q
            
            y.push(s_t)
            time += dt
    
    return { t, y, fs, I_envelope: I_env, Q_envelope: Q_env }
```

### JavaScript Implementation
```javascript
function generateWaveform(IQSymbols, fc, symbolDurMs, samplesPerSymbol=null) {
  // Step 1: Convert symbol duration to seconds
  const Ts = symbolDurMs / 1000;
  
  // Step 2: Calculate optimal sampling frequency
  const fs_min = Math.max(10 * fc, 64 / Ts);
  const fs = Math.pow(2, Math.ceil(Math.log2(fs_min)));
  
  // Step 3: Calculate samples per symbol and time step
  const sps = Math.round(fs * Ts);
  const dt = 1 / fs;
  
  // Step 4: Initialize output arrays
  const t = [];          // Time axis
  const y = [];          // Modulated signal
  const I_envelope = []; // I component (step function)
  const Q_envelope = []; // Q component (step function)
  
  let time = 0;
  
  // Step 5: Generate waveform sample by sample
  for (let k = 0; k < IQSymbols.length; k++) {
    const I = IQSymbols[k].I;
    const Q = IQSymbols[k].Q;
    
    // Generate sps samples for this symbol
    for (let n = 0; n < sps; n++) {
      t.push(time);
      I_envelope.push(I);
      Q_envelope.push(Q);
      
      // QAM modulation: s(t) = I*cos(2πf_c*t) - Q*sin(2πf_c*t)
      const carrier_I = I * Math.cos(2 * Math.PI * fc * time);
      const carrier_Q = -Q * Math.sin(2 * Math.PI * fc * time);
      const modulated = carrier_I + carrier_Q;
      
      y.push(modulated);
      time += dt;
    }
  }
  
  // Step 6: Return data with metadata
  return { t, y, fs, I_envelope, Q_envelope };
}
```

---

## 3. CRO RENDERING - MULTI-COMPONENT DISPLAY

### Display Mode Selector (HTML)
```html
<div style="...">
  <div>CH2 Display:</div>
  <input type="radio" name="ch2Display" value="qam" checked> QAM Signal
  <input type="radio" name="ch2Display" value="carrier"> Carrier
  <input type="radio" name="ch2Display" value="i"> I-Component
  <input type="radio" name="ch2Display" value="q"> Q-Component
</div>
```

### Mode Detection & Rendering (JavaScript)
```javascript
if (ch2DisplayMode === 'carrier') {
  // Render pure carrier signal
  const fc = Number(carrierFreq.value) || 2000;
  y2 = Math.cos(2 * Math.PI * fc * (idx / fs));
  
} else if (ch2DisplayMode === 'i') {
  // Render I-component (step function during symbol)
  y2 = lastCleanWave.I_envelope[idx] || 0;
  
} else if (ch2DisplayMode === 'q') {
  // Render Q-component (step function during symbol)
  y2 = lastCleanWave.Q_envelope[idx] || 0;
  
} else {
  // Default: Full QAM signal
  y2 = (lastCleanWave.y[idx] * scale + offset);
}

// Clamp to display range to prevent distortion
y2 = Math.max(-5, Math.min(5, y2));
```

---

## 4. MATHEMATICAL RELATIONSHIPS

### 4-QAM Example
```
Bits → Symbols mapping:
  00 → I=+1, Q=+1
  01 → I=+1, Q=-1  
  10 → I=-1, Q=+1
  11 → I=-1, Q=-1

Waveform for single symbol (00):
  I(t) = +1 (constant for duration Ts)
  Q(t) = +1 (constant for duration Ts)
  fc = 2000 Hz (carrier frequency)
  
  s(t) = (+1)×cos(2π×2000×t) - (+1)×sin(2π×2000×t)
       = cos(2π×2000×t) - sin(2π×2000×t)
       = √2 × cos(2π×2000×t + 45°)
```

### 16-QAM Example
```
Bits → Symbols mapping (Gray coded):
  0000 → I=-3, Q=-3
  0001 → I=-3, Q=-1
  0011 → I=-3, Q=+1
  0010 → I=-3, Q=+3
  0110 → I=-1, Q=-3
  0111 → I=-1, Q=-1
  0101 → I=-1, Q=+1
  0100 → I=-1, Q=+3
  1100 → I=+1, Q=-3
  1101 → I=+1, Q=-1
  1111 → I=+1, Q=+1
  1110 → I=+1, Q=+3
  1010 → I=+3, Q=-3
  1011 → I=+3, Q=-1
  1001 → I=+3, Q=+1
  1000 → I=+3, Q=+3
```

---

## 5. VISUALIZATION: Before & After

### BEFORE (Old Implementation)
```
Sampling: fs = 64 / Ts (fixed samplesPerSymbol=64)
Problem: Doesn't guarantee Nyquist criterion

Waveform view (CH2):
  [Possibly aliased waveform depending on parameters]
  [May show wrong frequency content]
  [Could exhibit foldover distortion]

Example with high fc and low Ts:
  Aliasing artifacts visible
  Frequency doesn't match actual carrier
```

### AFTER (New Implementation)
```
Sampling: fs = 2^ceil(log2(max(10×fc, 64/Ts)))
Benefit: Always meets Nyquist with 10x margin

Waveform view (CH2) - QAM Mode:
  ├─ Clear modulation envelope visible
  ├─ I and Q components amplitude-modulate carrier
  ├─ Smooth, no aliasing
  └─ Correct frequency representation

Waveform view (CH2) - Carrier Mode:
  └─ Pure cos(2πf_c×t) at correct frequency

Waveform view (CH2) - I Mode:
  └─ Step function, constant per symbol period

Waveform view (CH2) - Q Mode:
  └─ Step function, constant per symbol period
```

---

## 6. DISPLAY AUTO-SCALING

### Volt/Div Calculation
```javascript
// Calculate peak amplitude of signal
let maxCH2 = 0;

if (ch2DisplayMode === 'carrier') {
  maxCH2 = 1.0;  // Carrier always ±1
  
} else if (ch2DisplayMode === 'i' && lastCleanWave.I_envelope) {
  // Find max I component value
  maxCH2 = Math.max(...I_envelope.map(v => Math.abs(v)));
  
} else if (ch2DisplayMode === 'q' && lastCleanWave.Q_envelope) {
  // Find max Q component value
  maxCH2 = Math.max(...Q_envelope.map(v => Math.abs(v)));
  
} else {
  // Full QAM - find max modulated value
  for (let i = 0; i < fs; i++) {
    const val = Math.abs((y[i] * scale + offset));
    maxCH2 = Math.max(maxCH2, val);
  }
}

// Add 20% headroom
maxCH2 *= 1.2;

// Calculate display scaling
if (maxCH2 > voltScale * 5) {
  voltScale = maxCH2 / 5;  // Rescale divisions
  SimulationEngine.voltPerDiv = voltScale;
}
```

### Result
✓ Waveforms never clip
✓ Full use of display area
✓ Dynamic adjustment to parameters
✓ 20% margin for transients

---

## 7. TESTING SCENARIOS

### Test 1: Basic 4-QAM
```
Parameters:
  Modulation: 4-QAM
  Bits: 100
  SNR: 20 dB
  Carrier Freq: 1000 Hz
  Symbol Rate: 1000 sym/s
  
Expected Result:
  CH1: Clean ~1V sine at 1000 Hz
  CH2 (QAM): Modulated sine with visible amplitude changes every 1ms
  CH2 (Carrier): Pure 1000 Hz sine
  CH2 (I-component): Step function alternating between ±1
  CH2 (Q-component): Step function alternating between ±1
```

### Test 2: High Carrier 16-QAM
```
Parameters:
  Modulation: 16-QAM
  Bits: 1000
  SNR: 20 dB
  Carrier Freq: 5000 Hz
  Symbol Rate: 2000 sym/s
  
Expected Result:
  CH1: Clean ~2V signal
  CH2 (QAM): Rapidly modulating waveform (clear modulation visible)
  CH2 (Carrier): 5000 Hz sine with no distortion
  CH2 (I): Step function with 4 levels: ±1, ±3
  CH2 (Q): Step function with 4 levels: ±1, ±3
```

### Test 3: Auto-Scaling
```
Parameters:
  - Adjust CH1 Amplitude slider through full range (0.1 → 5 V)
  - Observe CH1 display adjusts automatically
  - Observe VOLT/DIV value changes
  
Expected Result:
  - Waveform always fits in center of display
  - No clipping at any amplitude
  - VOLT/DIV scales appropriately
```

---

## 8. TROUBLESHOOTING REFERENCE

### Issue: Waveform appears distorted/aliased
**Diagnosis**: 
```javascript
// In browser console:
console.log("Sampling frequency:", lastCleanWave.fs);
console.log("Carrier frequency:", Number(document.getElementById('carrierFreq').value));
console.log("Ratio:", lastCleanWave.fs / Number(document.getElementById('carrierFreq').value));
```
Expected ratio should be > 10

### Issue: I/Q components not appearing
**Check**:
```javascript
// Verify I and Q envelopes exist
console.log(lastCleanWave.I_envelope.slice(0, 10));
console.log(lastCleanWave.Q_envelope.slice(0, 10));
```

### Issue: CH2 display blank
**Check**:
```javascript
// Verify lastCleanWave exists
console.log(lastCleanWave);
// Should show {t: [...], y: [...], fs: ..., I_envelope: [...], Q_envelope: [...]}
```

---

**Document Version**: 1.0
**Updated**: 2026-02-28
