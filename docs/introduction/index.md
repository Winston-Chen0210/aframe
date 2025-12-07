<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>尋隱者不遇 VR - 賈島 Premium</title>
    <script src="https://aframe.io/releases/1.4.2/aframe.min.js"></script>
    <style>
      /* 開始畫面樣式 */
      #startScreen {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: linear-gradient(135deg, #1a2a1a 0%, #2d4a3a 50%, #1a2a1a 100%);
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        z-index: 9999;
        font-family: 'Noto Serif TC', serif;
      }
      
      #startScreen h1 {
        color: #d4af37;
        font-size: 3.5em;
        margin-bottom: 20px;
        text-shadow: 3px 3px 6px rgba(0,0,0,0.5);
        letter-spacing: 15px;
      }
      
      #startScreen h2 {
        color: #a8c8a8;
        font-size: 1.5em;
        margin-bottom: 40px;
        letter-spacing: 8px;
      }
      
      #startScreen .poem-preview {
        color: #e8e0d0;
        font-size: 1.2em;
        line-height: 2.2;
        margin-bottom: 50px;
        text-align: center;
        letter-spacing: 5px;
      }
      
      #startBtn {
        padding: 20px 60px;
        font-size: 1.5em;
        background: linear-gradient(135deg, #8b4513 0%, #d4a574 50%, #8b4513 100%);
        color: #fff;
        border: 3px solid #d4af37;
        border-radius: 50px;
        cursor: pointer;
        font-family: 'Noto Serif TC', serif;
        letter-spacing: 8px;
        transition: all 0.3s ease;
        box-shadow: 0 8px 25px rgba(0,0,0,0.4);
      }
      
      #startBtn:hover {
        transform: scale(1.1);
        box-shadow: 0 12px 35px rgba(212, 175, 55, 0.5);
      }
      
      .decoration {
        position: absolute;
        font-size: 4em;
        color: #3a5a3a;
        opacity: 0.3;
      }
      
      .deco-top-left { top: 30px; left: 30px; }
      .deco-top-right { top: 30px; right: 30px; }
      .deco-bottom-left { bottom: 30px; left: 30px; }
      .deco-bottom-right { bottom: 30px; right: 30px; }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Serif+TC:wght@400;700&display=swap" rel="stylesheet">
    <script>
      // ===== 詩文漂浮動畫組件 =====
      AFRAME.registerComponent('poem-float', {
        schema: {
          amplitude: {type: 'number', default: 0.3},
          speed: {type: 'number', default: 1},
          offset: {type: 'number', default: 0},
          rotationRange: {type: 'number', default: 3}
        },
        init: function() {
          this.originalY = this.el.getAttribute('position').y;
          this.originalX = this.el.getAttribute('position').x;
        },
        tick: function(time) {
          var amplitude = this.data.amplitude;
          var speed = this.data.speed;
          var offset = this.data.offset;
          var rotRange = this.data.rotationRange;
          
          // Y軸上下漂浮
          var newY = this.originalY + Math.sin((time + offset) * 0.001 * speed) * amplitude;
          // X軸輕微左右搖擺
          var newX = this.originalX + Math.cos((time + offset) * 0.0008 * speed) * amplitude * 0.3;
          // Z軸輕微旋轉
          var rotZ = Math.sin((time + offset) * 0.0005 * speed) * rotRange;
          
          this.el.object3D.position.y = newY;
          this.el.object3D.position.x = newX;
          this.el.object3D.rotation.z = THREE.MathUtils.degToRad(rotZ);
        }
      });

      // ===== 雲霧飄動動畫組件 =====
      AFRAME.registerComponent('cloud-drift', {
        schema: {
          speed: {type: 'number', default: 0.5},
          range: {type: 'number', default: 0.5},
          offset: {type: 'number', default: 0}
        },
        init: function() {
          this.originalPos = {
            x: this.el.getAttribute('position').x,
            y: this.el.getAttribute('position').y,
            z: this.el.getAttribute('position').z
          };
        },
        tick: function(time) {
          var speed = this.data.speed;
          var range = this.data.range;
          var offset = this.data.offset;
          
          this.el.object3D.position.y = this.originalPos.y + Math.sin((time + offset) * 0.0008 * speed) * range;
          this.el.object3D.position.x = this.originalPos.x + Math.cos((time + offset) * 0.0005 * speed) * range * 0.5;
          this.el.object3D.position.z = this.originalPos.z + Math.sin((time + offset) * 0.0003 * speed) * range * 0.3;
        }
      });

      // ===== 光影呼吸效果 =====
      AFRAME.registerComponent('light-pulse', {
        schema: {
          minIntensity: {type: 'number', default: 0.4},
          maxIntensity: {type: 'number', default: 0.8},
          speed: {type: 'number', default: 1}
        },
        tick: function(time) {
          var min = this.data.minIntensity;
          var max = this.data.maxIntensity;
          var speed = this.data.speed;
          var intensity = min + (max - min) * (Math.sin(time * 0.0005 * speed) + 1) / 2;
          this.el.setAttribute('light', 'intensity', intensity);
        }
      });

      // ===== 粒子系統旋轉 =====
      AFRAME.registerComponent('particle-rotate', {
        schema: {
          speed: {type: 'number', default: 0.1}
        },
        tick: function(time) {
          this.el.object3D.rotation.y += this.data.speed * 0.0001;
        }
      });

      // ===== 開始體驗函數 =====
      function startExperience() {
        document.getElementById('startScreen').style.display = 'none';
        
        // 播放背景音樂
        var bgMusic = document.getElementById('bgMusic');
        if (bgMusic) {
          bgMusic.components.sound.playSound();
        }
        
        // 播放環境音效
        var ambientSound = document.getElementById('ambientSound');
        if (ambientSound) {
          ambientSound.components.sound.playSound();
        }
      }
    </script>
  </head>
  <body>
    <!-- ===== 開始畫面 ===== -->
    <div id="startScreen">
      <span class="decoration deco-top-left">☘</span>
      <span class="decoration deco-top-right">☘</span>
      <span class="decoration deco-bottom-left">🌿</span>
      <span class="decoration deco-bottom-right">🌿</span>
      
      <h1>尋隱者不遇</h1>
      <h2>唐 · 賈島</h2>
      <div class="poem-preview">
        松下問童子，言師採藥去。<br>
        只在此山中，雲深不知處。
      </div>
      <button id="startBtn" onclick="startExperience()">進入山林</button>
    </div>

    <!-- ===== VR 場景 ===== -->
    <a-scene fog="type: exponential; color: #D5DFD8; density: 0.018">
      
      <!-- ===== 音樂系統 ===== -->
      <!-- 中國風背景音樂 (使用免費音樂來源) -->
      <a-entity id="bgMusic" 
                sound="src: url(https://cdn.pixabay.com/download/audio/2022/02/23/audio_ea70ad08e0.mp3?filename=asian-ambient-atmosphere-19447.mp3); 
                       autoplay: false; 
                       loop: true; 
                       volume: 0.4;
                       positional: false">
      </a-entity>
      
      <!-- 自然環境音效 (風聲、鳥鳴) -->
      <a-entity id="ambientSound" 
                sound="src: url(https://cdn.pixabay.com/download/audio/2022/03/10/audio_4dab3c3f53.mp3?filename=birds-19624.mp3); 
                       autoplay: false; 
                       loop: true; 
                       volume: 0.25;
                       positional: false">
      </a-entity>

      <!-- ===== 天空 ===== -->
      <a-sky color="#B8CCC4"></a-sky>

      <!-- ===== 地面系統 ===== -->
      <a-plane rotation="-90 0 0" width="150" height="150" color="#4A5D23"></a-plane>
      <a-plane rotation="-90 0 0" width="150" height="150" color="#5A6E33" position="0 0.01 0"></a-plane>

      <!-- 石板路 -->
      <a-box position="0 0.04 0" width="2" height="0.08" depth="20" color="#8B7355"></a-box>
      <a-box position="0.6 0.04 0" width="0.8" height="0.08" depth="20" color="#A0826D"></a-box>
      <a-box position="-0.6 0.04 0" width="0.8" height="0.08" depth="20" color="#A0826D"></a-box>

      <!-- ===== 漂浮詩文系統 ===== -->
      <!-- 主標題 - 大幅漂浮 -->
      <a-entity position="0 6 -12" poem-float="amplitude: 0.4; speed: 0.8; offset: 0; rotationRange: 2">
        <a-text value="尋隱者不遇" 
                align="center" 
                color="#2F4F2F" 
                width="14"
                opacity="0.95">
        </a-text>
        <!-- 標題裝飾光暈 -->
        <a-sphere position="0 0 -0.5" radius="3" color="#FFFFFF" opacity="0.08"></a-sphere>
      </a-entity>
      
      <!-- 作者 -->
      <a-entity position="0 5.2 -12" poem-float="amplitude: 0.35; speed: 0.9; offset: 500; rotationRange: 1.5">
        <a-text value="唐 · 賈島" 
                align="center" 
                color="#4A6B42" 
                width="10"
                opacity="0.9">
        </a-text>
      </a-entity>
      
      <!-- 詩句第一句 - 左側漂浮 -->
      <a-entity position="-4 4.5 -10" poem-float="amplitude: 0.5; speed: 0.7; offset: 1000; rotationRange: 4">
        <a-text value="松下問童子" 
                align="center" 
                color="#3A5A3A" 
                width="8"
                opacity="0.92">
        </a-text>
        <a-sphere position="0 0 -0.3" radius="1.8" color="#E8F0E8" opacity="0.1"></a-sphere>
      </a-entity>
      
      <!-- 詩句第二句 - 右側漂浮 -->
      <a-entity position="4 4 -11" poem-float="amplitude: 0.45; speed: 0.75; offset: 1500; rotationRange: 3.5">
        <a-text value="言師採藥去" 
                align="center" 
                color="#3A5A3A" 
                width="8"
                opacity="0.92">
        </a-text>
        <a-sphere position="0 0 -0.3" radius="1.8" color="#E8F0E8" opacity="0.1"></a-sphere>
      </a-entity>
      
      <!-- 詩句第三句 - 左下漂浮 -->
      <a-entity position="-3.5 3.2 -9" poem-float="amplitude: 0.55; speed: 0.65; offset: 2000; rotationRange: 4.5">
        <a-text value="只在此山中" 
                align="center" 
                color="#3A5A3A" 
                width="8"
                opacity="0.92">
        </a-text>
        <a-sphere position="0 0 -0.3" radius="1.8" color="#E8F0E8" opacity="0.1"></a-sphere>
      </a-entity>
      
      <!-- 詩句第四句 - 右下漂浮（最重要的一句）
