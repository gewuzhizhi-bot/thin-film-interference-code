# thin-film-interference
薄膜干涉详细代码 见下：
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>等厚薄膜干涉模拟器</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    <style>
        .glass-effect {
            background: rgba(255, 255, 255, 0.7);
            backdrop-filter: blur(10px);
        }
        .slider-thumb::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 18px;
            height: 18px;
            border-radius: 50%;
            background: #3B82F6;
            cursor: pointer;
        }
        .btn-active {
            background-color: #1E40AF;
            color: white;
        }
        canvas {
            display: block;
        }
        .light-path {
            stroke-dasharray: 5, 3;
            stroke-width: 1;
            stroke: rgba(0, 0, 0, 0.5);
        }
        .rainbow {
            background: linear-gradient(90deg, #ff0000, #ff9900, #ffff00, #00ff00, #00ffff, #0000ff, #9900ff);
        }
        .glow {
            box-shadow: 0 0 15px rgba(255, 255, 255, 0.8);
        }
        .color-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            display: inline-block;
            margin: 0 2px;
        }
        .dark-mode-bg {
            background: #1a202c;
        }
        .thickness-indicator {
            position: relative;
            height: 8px;
            background: linear-gradient(90deg, #000, #444, #888, #ccc, #fff);
            border-radius: 4px;
            margin-top: 5px;
        }
        .thickness-marker {
            position: absolute;
            top: -5px;
            width: 2px;
            height: 18px;
            background: red;
            transform: translateX(-50%);
        }
        .thickness-label {
            position: absolute;
            top: -25px;
            font-size: 10px;
            transform: translateX(-50%);
            white-space: nowrap;
        }
        .fade-in {
            animation: fadeIn 0.3s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
        .transition-all-smooth {
            transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .speed-info {
            background-color: #f0f9ff;
            border-left: 4px solid #3b82f6;
            padding: 10px;
            margin-top: 10px;
            border-radius: 4px;
        }
        .speed-indicator {
            height: 4px;
            border-radius: 2px;
            margin: 8px 0;
            transition: width 0.3s;
        }
        .speed-label {
            font-size: 11px;
            color: #4b5563;
        }
        .parameter-graph {
            height: 100px;
            position: relative;
            border-left: 1px solid #ccc;
            border-bottom: 1px solid #ccc;
            margin-top: 10px;
        }
        .parameter-line {
            position: absolute;
            bottom: 0;
            height: 2px;
            background-color: #3b82f6;
            transition: width 0.5s ease-out;
        }
        .parameter-label {
            position: absolute;
            font-size: 10px;
            color: #6b7280;
            bottom: -20px;
        }
        .easing-indicator {
            height: 30px;
            background: linear-gradient(90deg, #f0f9ff, #dbeafe, #bfdbfe);
            border-radius: 4px;
            margin-top: 10px;
            position: relative;
            overflow: hidden;
        }
        .easing-ball {
            position: absolute;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: #3b82f6;
            top: 5px;
            transition: left 1.5s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .param-change-indicator {
            position: absolute;
            top: -25px;
            right: 0;
            background: #ef4444;
            color: white;
            font-size: 10px;
            padding: 2px 5px;
            border-radius: 4px;
            opacity: 0;
            transition: opacity 0.3s;
        }
        .param-change-indicator.active {
            opacity: 1;
        }
        .physics-correction {
            background: #fffbeb;
            border-left: 4px solid #f59e0b;
            padding: 10px;
            border-radius: 4px;
            margin-top: 10px;
        }
        .interference-pattern-title {
            font-size: 18px;
            font-weight: 600;
            color: #1e40af;
            text-align: center;
            margin-bottom: 10px;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen p-4">
    <div class="container mx-auto max-w-6xl">
        <header class="mb-8 text-center">
            <h1 class="text-3xl font-bold text-gray-800 mb-2">等厚薄膜干涉模拟器</h1>
            <p class="text-gray-600">2025物理实验竞赛使用 | 真实物理原理模拟</p>
            <div class="text-sm bg-blue-100 p-2 rounded-lg mt-2">
                <i class="fa fa-info-circle text-blue-500 mr-1"></i>
                精确模拟薄膜干涉现象，可调节厚度、折射率、入射角和光源类型
            </div>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <!-- 控制面板 -->
            <div class="lg:col-span-1 space-y-6">
                <div class="glass-effect rounded-xl p-6 shadow-lg">
                    <h2 class="text-xl font-semibold mb-4 flex items-center">
                        <i class="fa fa-sliders mr-2 text-blue-500"></i>参数控制
                    </h2>
                    
                    <div class="space-y-5">
                        <!-- 薄膜厚度 -->
                        <div class="relative">
                            <label class="block text-sm font-medium mb-1">薄膜厚度 (nm)</label>
                            <div class="flex items-center">
                                <input type="range" id="thickness" min="10" max="1000" value="300" 
                                    class="w-full h-2 bg-gray-200 rounded-lg slider-thumb">
                            </div>
                            <div class="flex justify-between text-xs mt-1">
                                <span>10nm</span>
                                <span id="thickness-value">300nm</span>
                                <span>1000nm</span>
                            </div>
                            <div id="thickness-indicator" class="thickness-indicator">
                                <div class="thickness-label" style="left: 0%;">黑色</div>
                                <div class="thickness-label" style="left: 10%;">紫色/蓝色</div>
                                <div class="thickness-label" style="left: 20%;">绿色</div>
                                <div class="thickness-label" style="left: 30%;">橙红色</div>
                                <div class="thickness-label" style="left: 50%;">重复</div>
                                <div id="thickness-marker" class="thickness-marker" style="left: 30%;"></div>
                            </div>
                            <div id="thickness-change" class="param-change-indicator">变化中</div>
                        </div>
                        
                        <!-- 波长（只在单色光模式下显示） -->
                        <div id="wavelength-control" class="transition-all-smooth relative">
                            <label class="block text-sm font-medium mb-1">波长 (nm)</label>
                            <div class="flex items-center">
                                <input type="range" id="wavelength" min="380" max="780" value="550" 
                                    class="w-full h-2 bg-gray-200 rounded-lg slider-thumb">
                            </div>
                            <div class="flex justify-between text-xs mt-1">
                                <span>380nm</span>
                                <span id="wavelength-value">550nm</span>
                                <span>780nm</span>
                            </div>
                            <div id="color-indicator" class="w-full h-3 mt-2 rounded-lg"></div>
                            <div id="wavelength-change" class="param-change-indicator">变化中</div>
                        </div>
                        
                        <!-- 折射率 -->
                        <div class="relative">
                            <label class="block text-sm font-medium mb-1">折射率</label>
                            <div class="flex items-center">
                                <input type="range" id="refractive-index" min="1.0" max="2.5" step="0.01" value="1.38" 
                                    class="w-full h-2 bg-gray-200 rounded-lg slider-thumb">
                            </div>
                            <div class="flex justify-between text-xs mt-1">
                                <span>1.0</span>
                                <span id="refractive-index-value">1.38</span>
                                <span>2.5</span>
                            </div>
                            <div id="refractive-change" class="param-change-indicator">变化中</div>
                        </div>
                        
                        <!-- 入射角 -->
                        <div class="relative">
                            <label class="block text-sm font-medium mb-1">入射角 (度)</label>
                            <div class="flex items-center">
                                <input type="range" id="incidence-angle" min="0" max="80" step="1" value="30" 
                                    class="w-full h-2 bg-gray-200 rounded-lg slider-thumb">
                            </div>
                            <div class="flex justify-between text-xs mt-1">
                                <span>0°</span>
                                <span id="incidence-angle-value">30°</span>
                                <span>80°</span>
                            </div>
                            <div id="angle-change" class="param-change-indicator">变化中</div>
                        </div>
                        
                        <!-- 光源类型 -->
                        <div>
                            <label class="block text-sm font-medium mb-2">光源类型</label>
                            <div class="flex space-x-2">
                                <button id="monochromatic-btn" class="flex-1 py-2 px-4 rounded-lg transition btn-active">
                                    <i class="fa fa-circle-o mr-2"></i>单色光
                                </button>
                                <button id="white-light-btn" class="flex-1 py-2 px-4 rounded-lg transition bg-white">
                                    <i class="fa fa-sun-o mr-2"></i>白光
                                </button>
                            </div>
                        </div>
                        
                        <!-- 动态模拟控制 -->
                        <div class="glass-effect rounded-lg p-4 mt-4">
                            <h3 class="text-sm font-medium mb-2">动态模拟控制</h3>
                            
                            <div class="grid grid-cols-2 gap-2 mb-3">
                                <label class="flex items-center space-x-2">
                                    <input type="checkbox" id="animate-thickness" class="w-4 h-4 text-blue-500 rounded" checked>
                                    <span class="text-sm">薄膜厚度</span>
                                </label>
                                <label class="flex items-center space-x-2">
                                    <input type="checkbox" id="animate-wavelength" class="w-4 h-4 text-blue-500 rounded" checked>
                                    <span class="text-sm">波长</span>
                                </label>
                                <label class="flex items-center space-x-2">
                                    <input type="checkbox" id="animate-refractive-index" class="w-4 h-4 text-blue-500 rounded" checked>
                                    <span class="text-sm">折射率</span>
                                </label>
                                <label class="flex items-center space-x-2">
                                    <input type="checkbox" id="animate-incidence-angle" class="w-4 h-4 text-blue-500 rounded" checked>
                                    <span class="text-sm">入射角</span>
                                </label>
                            </div>
                            
                            <div class="mb-3">
                                <label class="block text-sm font-medium mb-1">模拟速度</label>
                                <input type="range" id="animation-speed" min="1" max="10" value="5" 
                                    class="w-full h-2 bg-gray-200 rounded-lg slider-thumb">
                                <div class="flex justify-between text-xs mt-1">
                                    <span>慢</span>
                                    <span id="speed-value">中速</span>
                                    <span>快</span>
                                </div>
                            </div>
                            
                            <!-- 参数速度指示器 -->
                            <div class="mt-4 bg-blue-50 p-3 rounded-lg">
                                <h4 class="text-sm font-medium mb-2 flex items-center">
                                    <i class="fa fa-tachometer mr-2 text-blue-600"></i>参数变化速度
                                </h4>
                                <div class="space-y-3">
                                    <div>
                                        <div class="flex justify-between">
                                            <span class="text-xs">厚度变化</span>
                                            <span id="thickness-speed" class="text-xs font-mono">0.0</span>
                                        </div>
                                        <div id="thickness-speed-bar" class="speed-indicator bg-blue-200" style="width: 0%"></div>
                                    </div>
                                    <div>
                                        <div class="flex justify-between">
                                            <span class="text-xs">波长变化</span>
                                            <span id="wavelength-speed" class="text-xs font-mono">0.0</span>
                                        </div>
                                        <div id="wavelength-speed-bar" class="speed-indicator bg-blue-300" style="width: 0%"></div>
                                    </div>
                                    <div>
                                        <div class="flex justify-between">
                                            <span class="text-xs">折射率变化</span>
                                            <span id="refractive-speed" class="text-xs font-mono">0.0</span>
                                        </div>
                                        <div id="refractive-speed-bar" class="speed-indicator bg-blue-400" style="width: 0%"></div>
                                    </div>
                                    <div>
                                        <div class="flex justify-between">
                                            <span class="text-xs">入射角变化</span>
                                            <span id="angle-speed" class="text-xs font-mono">0.0</span>
                                        </div>
                                        <div id="angle-speed-bar" class="speed-indicator bg-blue-500" style="width: 0%"></div>
                                    </div>
                                </div>
                            </div>
                            
                            <!-- 缓动效果指示器 -->
                            <div class="easing-indicator mt-4">
                                <div id="easing-ball" class="easing-ball" style="left: 5px"></div>
                            </div>
                            
                            <div class="flex space-x-2 mt-4">
                                <button id="reset-btn" class="flex-1 py-2 px-4 bg-gray-200 rounded-lg transition hover:bg-gray-300">
                                    <i class="fa fa-refresh mr-2"></i>重置
                                </button>
                                <button id="play-btn" class="flex-1 py-2 px-4 bg-blue-500 text-white rounded-lg transition hover:bg-blue-600">
                                    <i class="fa fa-play mr-2"></i>开始模拟
                                </button>
                            </div>
                            
                            <div class="speed-info mt-4">
                                <p class="text-sm text-blue-800">
                                    <i class="fa fa-info-circle mr-1"></i> 
                                    各参数速度不同 可以通过调节模拟速度来观察图像变化
                                </p>
                            </div>
                        </div>
                    </div>
                </div>
                
                <!-- 干涉数据 -->
                <div class="glass-effect rounded-xl p-6 shadow-lg">
                    <h2 class="text-xl font-semibold mb-4 flex items-center">
                        <i class="fa fa-calculator mr-2 text-blue-500"></i>干涉数据
                    </h2>
                    <div class="space-y-3">
                        <div class="bg-white rounded-lg p-3 shadow">
                            <div class="flex justify-between">
                                <span class="text-sm">光程差:</span>
                                <span id="optical-path-difference" class="font-mono">0nm</span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-sm">相位差:</span>
                                <span id="phase-difference" class="font-mono">0rad</span>
                            </div>
                        </div>
                        <div class="bg-white rounded-lg p-3 shadow">
                            <div class="flex justify-between">
                                <span class="text-sm">干涉条件:</span>
                                <span id="interference-condition" class="font-mono">相长干涉</span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-sm">干涉级次:</span>
                                <span id="interference-order" class="font-mono">m=0</span>
                            </div>
                        </div>
                        <div class="bg-white rounded-lg p-3 shadow">
                            <div class="text-sm mb-2">当前波长状态:</div>
                            <div id="wavelength-status" class="text-xs">
                                <div class="flex items-center mb-1">
                                    <span class="color-dot bg-red-500"></span>
                                    <span>红: 相长干涉</span>
                                </div>
                                <div class="flex items-center mb-1">
                                    <span class="color-dot bg-green-500"></span>
                                    <span>绿: 相消干涉</span>
                                </div>
                                <div class="flex items-center">
                                    <span class="color-dot bg-blue-500"></span>
                                    <span>蓝: 中间状态</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            
            <!-- 可视化区域 -->
            <div class="lg:col-span-2 space-y-6">
                <!-- 干涉示意图 -->
                <div class="glass-effect rounded-xl p-6 shadow-lg h-[400px]">
                    <h2 class="text-xl font-semibold mb-4 flex items-center">
                        <i class="fa fa-eye mr-2 text-blue-500"></i>干涉示意图
                    </h2>
                    <div class="h-[300px]">
                        <canvas id="interferenceCanvas"></canvas>
                    </div>
                </div>
                
                <!-- 干涉图样 -->
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div class="glass-effect rounded-xl p-6 shadow-lg h-[300px]">
                        <div class="interference-pattern-title">上表面干涉图样</div>
                        <div class="h-[200px]">
                            <canvas id="upperSurfaceCanvas"></canvas>
                        </div>
                    </div>
                    <div class="glass-effect rounded-xl p-6 shadow-lg h-[300px]">
                        <div class="interference-pattern-title">下表面干涉图样</div>
                        <div class="h-[200px]">
                            <canvas id="lowerSurfaceCanvas"></canvas>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        
        <!-- 白光干涉原理说明 -->
        <div class="glass-effect rounded-xl p-6 shadow-lg mt-6">
            <h2 class="text-xl font-semibold mb-4 flex items-center">
                <i class="fa fa-lightbulb-o mr-2 text-yellow-500"></i>物理原理说明
            </h2>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div class="bg-white rounded-lg p-4 shadow">
                    <h3 class="font-medium mb-2">1. 等厚干涉原理</h3>
                    <p class="text-sm text-gray-600">当平行光入射到薄膜上表面时，一部分光在上表面反射，另一部分光折射进入薄膜，在下表面反射后再次折射出薄膜。</p>
                    <div class="mt-2 flex justify-center">
                        <i class="fa fa-arrow-down text-blue-500 text-2xl"></i>
                    </div>
                </div>
                <div class="bg-white rounded-lg p-4 shadow">
                    <h3 class="font-medium mb-2">2. 光程差计算</h3>
                    <p class="text-sm text-gray-600">光程差 δ = 2n·d·cosθ，其中n为折射率，d为薄膜厚度，θ为折射角。当光程差满足特定条件时产生相长或相消干涉。</p>
                    <div class="mt-2 flex justify-center">
                        <i class="fa fa-calculator text-blue-500 text-2xl"></i>
                    </div>
                </div>
                <div class="bg-white rounded-lg p-4 shadow">
                    <h3 class="font-medium mb-2">3. 颜色形成</h3>
                    <p class="text-sm text-gray-600">白光干涉时，不同波长的光满足干涉条件不同，上表面形成特定颜色，下表面形成互补色。</p>
                    <div class="w-full h-3 mt-2 rounded-lg rainbow"></div>
                </div>
            </div>
            <div class="mt-6 bg-white rounded-lg p-4 shadow">
                <h3 class="font-medium mb-2">薄膜厚度与干涉颜色的关系</h3>
                <table class="w-full text-sm">
                    <thead>
                        <tr class="bg-gray-100">
                            <th class="border p-2">厚度 (nm)</th>
                            <th class="border p-2">上表面干涉图样</th>
                            <th class="border p-2">下表面干涉图样</th>
                            <th class="border p-2">物理原理</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td class="border p-2">薄 (d → 0)</td>
                            <td class="border p-2 bg-black text-white">黑色/深灰</td>
                            <td class="border p-2 bg-white">白色/亮色</td>
                            <td class="border p-2">所有波长均满足相消干涉 → 下表面透射光中所有波长相长</td>
                        </tr>
                        <tr>
                            <td class="border p-2">d ≈ 100</td>
                            <td class="border p-2" style="background: linear-gradient(90deg, #9400d3, #0000ff)"><span class="text-white">紫色/蓝色</span></td>
                            <td class="border p-2" style="background: linear-gradient(90deg, #adff2f, #ffff00)">黄绿色</td>
                            <td class="border p-2">短波长(蓝紫)上表面反射相长 → 下表面透射相消</td>
                        </tr>
                        <tr>
                            <td class="border p-2">d ≈ 200</td>
                            <td class="border p-2 bg-green-500 text-white">绿色</td>
                            <td class="border p-2" style="background: linear-gradient(90deg, #ff00ff, #ff1493)">品红色</td>
                            <td class="border p-2">绿光上表面反射相长 → 下表面透射相消</td>
                        </tr>
                        <tr>
                            <td class="border p-2">d ≈ 300</td>
                            <td class="border p-2" style="background: linear-gradient(90deg, #ff4500, #ff0000)"><span class="text-white">橙红色</span></td>
                            <td class="border p-2" style="background: linear-gradient(90deg, #00ffff, #00ced1)">蓝绿色</td>
                            <td class="border p-2">红光上表面反射相长 → 下表面透射相消</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <script>
        const interferenceCanvas = document.getElementById('interferenceCanvas');
        const upperSurfaceCanvas = document.getElementById('upperSurfaceCanvas');
        const lowerSurfaceCanvas = document.getElementById('lowerSurfaceCanvas');
        
        function resizeCanvases() {
            const interferenceContainer = interferenceCanvas.parentElement;
            const upperContainer = upperSurfaceCanvas.parentElement;
            const lowerContainer = lowerSurfaceCanvas.parentElement;
            
            interferenceCanvas.width = interferenceContainer.clientWidth;
            interferenceCanvas.height = interferenceContainer.clientHeight;
            
            upperSurfaceCanvas.width = upperContainer.clientWidth;
            upperSurfaceCanvas.height = upperContainer.clientHeight;
            
            lowerSurfaceCanvas.width = lowerContainer.clientWidth;
            lowerSurfaceCanvas.height = lowerContainer.clientHeight;
            
            updateSimulation();
        }
        
        window.addEventListener('load', () => {
            resizeCanvases();
            updateSimulation();
            animateEasingBall(); 
        });
        window.addEventListener('resize', resizeCanvases);
        
        const thicknessSlider = document.getElementById('thickness');
        const wavelengthSlider = document.getElementById('wavelength');
        const refractiveIndexSlider = document.getElementById('refractive-index');
        const incidenceAngleSlider = document.getElementById('incidence-angle');
        const monochromaticBtn = document.getElementById('monochromatic-btn');
        const whiteLightBtn = document.getElementById('white-light-btn');
        const thicknessMarker = document.getElementById('thickness-marker');
        const wavelengthControl = document.getElementById('wavelength-control');
        
        const animateThickness = document.getElementById('animate-thickness');
        const animateWavelength = document.getElementById('animate-wavelength');
        const animateRefractiveIndex = document.getElementById('animate-refractive-index');
        const animateIncidenceAngle = document.getElementById('animate-incidence-angle');
        const animationSpeedSlider = document.getElementById('animation-speed');
        const playBtn = document.getElementById('play-btn');
        const resetBtn = document.getElementById('reset-btn');
        const speedValue = document.getElementById('speed-value');
        
        const thicknessValue = document.getElementById('thickness-value');
        const wavelengthValue = document.getElementById('wavelength-value');
        const refractiveIndexValue = document.getElementById('refractive-index-value');
        const incidenceAngleValue = document.getElementById('incidence-angle-value');
        
        const opticalPathDifference = document.getElementById('optical-path-difference');
        const phaseDifference = document.getElementById('phase-difference');
        const interferenceCondition = document.getElementById('interference-condition');
        const interferenceOrder = document.getElementById('interference-order');
        const colorIndicator = document.getElementById('color-indicator');
        
        const thicknessSpeed = document.getElementById('thickness-speed');
        const wavelengthSpeed = document.getElementById('wavelength-speed');
        const refractiveSpeed = document.getElementById('refractive-speed');
        const angleSpeed = document.getElementById('angle-speed');
        const thicknessSpeedBar = document.getElementById('thickness-speed-bar');
        const wavelengthSpeedBar = document.getElementById('wavelength-speed-bar');
        const refractiveSpeedBar = document.getElementById('refractive-speed-bar');
        const angleSpeedBar = document.getElementById('angle-speed-bar');
        
        const thicknessChangeIndicator = document.getElementById('thickness-change');
        const wavelengthChangeIndicator = document.getElementById('wavelength-change');
        const refractiveChangeIndicator = document.getElementById('refractive-change');
        const angleChangeIndicator = document.getElementById('angle-change');
        
        const easingBall = document.getElementById('easing-ball');
        
        let isWhiteLight = false;
        let currentColor = [0, 0, 0];
        let isPlaying = false;
        let animationId = null;
        let lastTimestamp = 0;
        let lastThickness = 300; 
        let lastWavelength = 550; 
        
        thicknessSlider.addEventListener('input', updateSimulation);
        wavelengthSlider.addEventListener('input', updateSimulation);
        refractiveIndexSlider.addEventListener('input', updateSimulation);
        incidenceAngleSlider.addEventListener('input', updateSimulation);
        animationSpeedSlider.addEventListener('input', updateSpeedDisplay);
        
        monochromaticBtn.addEventListener('click', () => {
            isWhiteLight = false;
            monochromaticBtn.classList.add('btn-active');
            whiteLightBtn.classList.remove('btn-active');
            
            wavelengthControl.style.display = 'block';
            setTimeout(() => {
                wavelengthControl.classList.remove('opacity-0');
            }, 10);
            
            updateSimulation();
        });
        
        whiteLightBtn.addEventListener('click', () => {
            isWhiteLight = true;
            whiteLightBtn.classList.add('btn-active');
            monochromaticBtn.classList.remove('btn-active');
            
            wavelengthControl.classList.add('opacity-0');
            setTimeout(() => {
                wavelengthControl.style.display = 'none';
            }, 300);
            
            updateSimulation();
        });
        
        playBtn.addEventListener('click', toggleAnimation);
        resetBtn.addEventListener('click', resetSimulation);
        
        function updateSpeedDisplay() {
            const speed = parseInt(animationSpeedSlider.value);
            let speedText;
            if (speed <= 3) speedText = '慢速';
            else if (speed <= 7) speedText = '中速';
            else speedText = '快速';
            speedValue.textContent = speedText;
        }
        
        function animateEasingBall() {
            easingBall.style.left = '5px';
            setTimeout(() => {
                easingBall.style.left = 'calc(100% - 25px)';
            }, 100);
            
            setTimeout(() => {
                easingBall.style.left = '5px';
                setTimeout(animateEasingBall, 100);
            }, 1600);
        }
        
        function updateSpeedIndicators(deltaTime) {
            const speedFactor = (0.05 + (animationSpeedSlider.value / 10) * 0.9);
            
            const thicknessSpeedVal = 0.15 * speedFactor * deltaTime;
            const wavelengthSpeedVal = 0.08 * speedFactor * deltaTime;
            const refractiveSpeedVal = 0.003 * speedFactor * deltaTime;
            const angleSpeedVal = 0.15 * speedFactor * deltaTime;
            
            thicknessSpeed.textContent = thicknessSpeedVal.toFixed(4);
            wavelengthSpeed.textContent = wavelengthSpeedVal.toFixed(4);
            refractiveSpeed.textContent = refractiveSpeedVal.toFixed(6);
            angleSpeed.textContent = angleSpeedVal.toFixed(4);
            
            const maxSpeed = Math.max(thicknessSpeedVal, wavelengthSpeedVal, refractiveSpeedVal*1000, angleSpeedVal);
            
            thicknessSpeedBar.style.width = (thicknessSpeedVal / maxSpeed * 100) + '%';
            wavelengthSpeedBar.style.width = (wavelengthSpeedVal / maxSpeed * 100) + '%';
            refractiveSpeedBar.style.width = (refractiveSpeedVal*1000 / maxSpeed * 100) + '%';
            angleSpeedBar.style.width = (angleSpeedVal / maxSpeed * 100) + '%';
        }
        
        function updateParamChangeIndicators() {
            thicknessChangeIndicator.classList.toggle('active', animateThickness.checked);
            wavelengthChangeIndicator.classList.toggle('active', animateWavelength.checked && !isWhiteLight);
            refractiveChangeIndicator.classList.toggle('active', animateRefractiveIndex.checked);
            angleChangeIndicator.classList.toggle('active', animateIncidenceAngle.checked);
        }
        
        function toggleAnimation() {
            if (isPlaying) {
                stopAnimation();
                playBtn.innerHTML = '<i class="fa fa-play mr-2"></i>开始模拟';
            } else {
                startAnimation();
                playBtn.innerHTML = '<i class="fa fa-pause mr-2"></i>暂停模拟';
            }
            isPlaying = !isPlaying;
            updateParamChangeIndicators();
        }
        
        function startAnimation() {
            lastTimestamp = 0;
            animationId = requestAnimationFrame(animate);
        }
        
        function stopAnimation() {
            if (animationId) {
                cancelAnimationFrame(animationId);
                animationId = null;
            }
        }
        
        function animate(timestamp) {
            if (!lastTimestamp) lastTimestamp = timestamp;
            const deltaTime = timestamp - lastTimestamp;
            lastTimestamp = timestamp;
            
            updateSpeedIndicators(deltaTime);
            
            const speedFactor = (0.05 + (animationSpeedSlider.value / 10) * 0.9);
            
            if (animateThickness.checked) {
                const thickness = parseFloat(thicknessSlider.value);
                const newThickness = thickness + deltaTime * 0.15 * speedFactor;
                thicknessSlider.value = newThickness > 1000 ? 10 : newThickness < 10 ? 1000 : newThickness;
            }
            
            if (animateWavelength.checked && !isWhiteLight) {
                const wavelength = parseFloat(wavelengthSlider.value);
                const newWavelength = wavelength + deltaTime * 0.08 * speedFactor;
                wavelengthSlider.value = newWavelength > 780 ? 380 : newWavelength < 380 ? 780 : newWavelength;
            }
            
            if (animateRefractiveIndex.checked) {
                const refractiveIndex = parseFloat(refractiveIndexSlider.value);
                const newRefractiveIndex = refractiveIndex + deltaTime * 0.003 * speedFactor;
                refractiveIndexSlider.value = newRefractiveIndex > 2.5 ? 1.0 : newRefractiveIndex < 1.0 ? 2.5 : newRefractiveIndex;
            }
            
            if (animateIncidenceAngle.checked) {
                const incidenceAngle = parseFloat(incidenceAngleSlider.value);
                const newIncidenceAngle = incidenceAngle + deltaTime * 0.15 * speedFactor;
                if (newIncidenceAngle > 80) {
                    incidenceAngleSlider.value = 0;
                } else if (newIncidenceAngle < 0) {
                    incidenceAngleSlider.value = 80;
                } else {
                    incidenceAngleSlider.value = newIncidenceAngle;
                }
            }
            
            updateSimulation();
            
            animationId = requestAnimationFrame(animate);
        }
        

        function resetSimulation() {
            stopAnimation();
            isPlaying = false;
            playBtn.innerHTML = '<i class="fa fa-play mr-2"></i>开始模拟';
            

            thicknessSlider.value = 300;
            wavelengthSlider.value = 550;
            refractiveIndexSlider.value = 1.38;
            incidenceAngleSlider.value = 30;
            animationSpeedSlider.value = 5;
            

            animateThickness.checked = true;
            animateWavelength.checked = true;
            animateRefractiveIndex.checked = true;
            animateIncidenceAngle.checked = true;
            
            isWhiteLight = false;
            monochromaticBtn.classList.add('btn-active');
            whiteLightBtn.classList.remove('btn-active');
            
            wavelengthControl.style.display = 'block';
            wavelengthControl.classList.remove('opacity-0');
            
            updateSpeedDisplay();
            updateSimulation();
            updateParamChangeIndicators();
        }
        

        function updateSimulation() {
            thicknessValue.textContent = thicknessSlider.value + 'nm';
            wavelengthValue.textContent = wavelengthSlider.value + 'nm';
            refractiveIndexValue.textContent = refractiveIndexSlider.value;
            incidenceAngleValue.textContent = incidenceAngleSlider.value + '°';
            
            const thickness = parseFloat(thicknessSlider.value);
            thicknessMarker.style.left = (thickness / 1000 * 100) + '%';
            
            if (!isWhiteLight) {
                updateColorIndicator();
            }
            
            updateInterferenceData();
            
            drawInterferenceDiagram();
            drawInterferencePatterns();
            
            lastThickness = parseFloat(thicknessSlider.value);
            lastWavelength = parseFloat(wavelengthSlider.value);
        }
        
        function updateColorIndicator() {
            const wavelength = parseInt(wavelengthSlider.value);
            currentColor = wavelengthToRGB(wavelength);
            colorIndicator.style.backgroundColor = `rgb(${currentColor.join(',')})`;
        }
        
        function wavelengthToRGB(wavelength) {
            let r, g, b;
            
            if (wavelength >= 380 && wavelength < 440) {
                r = -1 * (wavelength - 440) / (440 - 380);
                g = 0;
                b = 1;
            } else if (wavelength >= 440 && wavelength < 490) {
                r = 0;
                g = (wavelength - 440) / (490 - 440);
                b = 1;
            } else if (wavelength >= 490 && wavelength < 510) {
                r = 0;
                g = 1;
                b = -1 * (wavelength - 510) / (510 - 490);
            } else if (wavelength >= 510 && wavelength < 580) {
                r = (wavelength - 510) / (580 - 510);
                g = 1;
                b = 0;
            } else if (wavelength >= 580 && wavelength < 645) {
                r = 1;
                g = -1 * (wavelength - 645) / (645 - 580);
                b = 0;
            } else if (wavelength >= 645 && wavelength <= 780) {
                r = 1;
                g = 0;
                b = 0;
            } else {
                r = 0;
                g = 0;
                b = 0;
            }
            
            let factor;
            if (wavelength >= 380 && wavelength < 420) {
                factor = 0.3 + 0.7 * (wavelength - 380) / (420 - 380);
            } else if (wavelength >= 420 && wavelength < 700) {
                factor = 1.0;
            } else if (wavelength >= 700 && wavelength <= 780) {
                factor = 0.3 + 0.7 * (780 - wavelength) / (780 - 700);
            } else {
                factor = 0.0;
            }
            
            return [
                Math.round(255 * Math.pow(r * factor, 0.8)),
                Math.round(255 * Math.pow(g * factor, 0.8)),
                Math.round(255 * Math.pow(b * factor, 0.8))
            ];
        }

        function updateInterferenceData() {
            const thickness = parseFloat(thicknessSlider.value);
            const wavelength = parseFloat(wavelengthSlider.value);
            const refractiveIndex = parseFloat(refractiveIndexSlider.value);
            const incidenceAngle = parseFloat(incidenceAngleSlider.value) * Math.PI / 180;
            
            const refractionAngle = Math.asin(Math.sin(incidenceAngle) / refractiveIndex);
            
            const opticalPathDiff = 2 * refractiveIndex * thickness * Math.cos(refractionAngle);
            
            const phaseDiff = (2 * Math.PI * opticalPathDiff) / wavelength;
            
            const order = opticalPathDiff / wavelength;
            
            opticalPathDifference.textContent = opticalPathDiff.toFixed(2) + 'nm';
            phaseDifference.textContent = phaseDiff.toFixed(2) + 'rad';
            interferenceOrder.textContent = 'm=' + order.toFixed(2);
            
            if (Math.abs(Math.round(order) - order) < 0.1) {
                interferenceCondition.textContent = '相长干涉';
                interferenceCondition.style.color = '#10B981';
            } else if (Math.abs(Math.round(order + 0.5) - (order + 0.5)) < 0.1) {
                interferenceCondition.textContent = '相消干涉';
                interferenceCondition.style.color = '#EF4444'; 
            } else {
                interferenceCondition.textContent = '中间状态';
                interferenceCondition.style.color = '#6B7280'; 
            }
        }
        
        function drawInterferenceDiagram() {
            const ctx = interferenceCanvas.getContext('2d');
            const width = interferenceCanvas.width;
            const height = interferenceCanvas.height;
            

            ctx.clearRect(0, 0, width, height);
            

            if (isWhiteLight) {
                ctx.fillStyle = '#1a202c'; 
                ctx.fillRect(0, 0, width, height);
            } else {
                ctx.fillStyle = 'white';
                ctx.fillRect(0, 0, width, height);
            }
            
            const thickness = parseFloat(thicknessSlider.value) / 1000 * height * 0.5;
            const centerY = height / 2;
            const filmTop = centerY - thickness / 2;
            const filmBottom = centerY + thickness / 2;
            const filmLeft = width * 0.2;
            const filmRight = width * 0.8;
            
            
            ctx.fillStyle = 'rgba(100, 200, 255, 0.3)';
            ctx.fillRect(filmLeft, filmTop, filmRight - filmLeft, thickness);
            ctx.strokeStyle = 'rgba(0, 0, 100, 0.8)';
            ctx.lineWidth = 2;
            ctx.strokeRect(filmLeft, filmTop, filmRight - filmLeft, thickness);
            
           
            const incidenceAngle = parseFloat(incidenceAngleSlider.value) * Math.PI / 180;
            const refractiveIndex = parseFloat(refractiveIndexSlider.value);
            const refractionAngle = Math.asin(Math.sin(incidenceAngle) / refractiveIndex);
            
            
            const incidentX = filmLeft;
            const incidentY = filmTop;
            
            const inRayLength = width * 0.3;
            const inStartX = incidentX - inRayLength * Math.cos(incidenceAngle);
            const inStartY = incidentY - inRayLength * Math.sin(incidenceAngle);
            
            if (isWhiteLight) {
                drawLightRay(ctx, inStartX, inStartY, incidentX, incidentY, 'white');
            } else {
                drawLightRay(ctx, inStartX, inStartY, incidentX, incidentY, `rgb(${currentColor.join(',')})`);
            }
            
            const reflRayLength = width * 0.3;
            const reflEndX = incidentX + reflRayLength * Math.cos(incidenceAngle);
            const reflEndY = incidentY - reflRayLength * Math.sin(incidenceAngle);
            
            if (isWhiteLight) {
               
                const thickness = parseFloat(thicknessSlider.value);
                let color;
                if (thickness < 50) color = 'black';
                else if (thickness < 150) color = 'linear-gradient(90deg, #9400d3, #0000ff)';
                else if (thickness < 250) color = '#00ff00';
                else color = 'linear-gradient(90deg, #ff4500, #ff0000)';
                
                drawColoredRay(ctx, incidentX, incidentY, reflEndX, reflEndY, color);
            } else {
                drawLightRay(ctx, incidentX, incidentY, reflEndX, reflEndY, `rgb(${currentColor.join(',')})`);
            }
            
            const refrRayLength = thickness / Math.cos(refractionAngle);
            const refrEndX = incidentX + refrRayLength * Math.sin(refractionAngle);
            const refrEndY = filmBottom;
            
            if (isWhiteLight) {
                drawLightRay(ctx, incidentX, incidentY, refrEndX, refrEndY, 'white');
            } else {
                drawLightRay(ctx, incidentX, incidentY, refrEndX, refrEndY, `rgb(${currentColor.join(',')})`);
            }
            
            const bottomReflEndX = refrEndX + thickness * Math.tan(refractionAngle);
            const bottomReflEndY = filmTop;
            
            if (isWhiteLight) {
                drawLightRay(ctx, refrEndX, refrEndY, bottomReflEndX, bottomReflEndY, 'white');
            } else {
                drawLightRay(ctx, refrEndX, refrEndY, bottomReflEndX, bottomReflEndY, `rgb(${currentColor.join(',')})`);
            }
            
            const secondReflEndX = bottomReflEndX + width * 0.3 * Math.cos(incidenceAngle);
            const secondReflEndY = bottomReflEndY - width * 0.3 * Math.sin(incidenceAngle);
            
            if (isWhiteLight) {
                const thickness = parseFloat(thicknessSlider.value);
                let color;
                if (thickness < 50) color = 'black';
                else if (thickness < 150) color = 'linear-gradient(90deg, #9400d3, #0000ff)';
                else if (thickness < 250) color = '#00ff00';
                else color = 'linear-gradient(90deg, #ff4500, #ff0000)';
                
                drawColoredRay(ctx, bottomReflEndX, bottomReflEndY, secondReflEndX, secondReflEndY, color);
            } else {
                drawLightRay(ctx, bottomReflEndX, bottomReflEndY, secondReflEndX, secondReflEndY, `rgb(${currentColor.join(',')})`);
            }
         
            const transExitX = refrEndX;
            const transExitY = refrEndY;

            const transRayLength = width * 0.3;
            const transEndX = transExitX + transRayLength * Math.cos(incidenceAngle);
            const transEndY = transExitY + transRayLength * Math.sin(incidenceAngle);
            
            if (isWhiteLight) {
                const thickness = parseFloat(thicknessSlider.value);
                let color;
                if (thickness < 50) color = 'white';
                else if (thickness < 150) color = 'linear-gradient(90deg, #adff2f, #ffff00)';
                else if (thickness < 250) color = 'linear-gradient(90deg, #ff00ff, #ff1493)';
                else color = 'linear-gradient(90deg, #00ffff, #00ced1)';
                
                drawColoredRay(ctx, transExitX, transExitY, transEndX, transEndY, color);
            } else {
                drawLightRay(ctx, transExitX, transExitY, transEndX, transEndY, `rgb(${currentColor.join(',')})`);
            }
            
            ctx.setLineDash([5, 3]);
            ctx.strokeStyle = isWhiteLight ? 'rgba(255, 255, 255, 0.5)' : 'rgba(0, 0, 0, 0.5)';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(incidentX, incidentY);
            ctx.lineTo(refrEndX, refrEndY);
            ctx.lineTo(bottomReflEndX, bottomReflEndY);
            ctx.stroke();
            ctx.setLineDash([]);
            
            ctx.font = '12px Arial';
            ctx.fillStyle = isWhiteLight ? 'white' : 'black';
            ctx.textAlign = 'center';
            ctx.fillText('反射光1', (incidentX + reflEndX)/2, (incidentY + reflEndY)/2 - 15);
            ctx.fillText('反射光2', (bottomReflEndX + secondReflEndX)/2, (bottomReflEndY + secondReflEndY)/2 - 15);
            ctx.fillText('透射光', (transExitX + transEndX)/2, (transExitY + transEndY)/2 + 15);
            

            const interferenceX = width * 0.8;
            const y1 = incidentY - (interferenceX - incidentX) * Math.tan(incidenceAngle);
            const y2 = bottomReflEndY - (interferenceX - bottomReflEndX) * Math.tan(incidenceAngle);
            const interferenceY = (y1 + y2) / 2;
            

            ctx.beginPath();
            ctx.arc(interferenceX, interferenceY, 8, 0, Math.PI*2);
            
            if (isWhiteLight) {

                const gradient = ctx.createRadialGradient(
                    interferenceX, interferenceY, 0,
                    interferenceX, interferenceY, 8
                );
                gradient.addColorStop(0, 'white');
                gradient.addColorStop(1, 'rgba(255, 255, 255, 0)');
                
                ctx.fillStyle = gradient;
            } else {
                ctx.fillStyle = 'red';
            }
            
            ctx.fill();
            ctx.fillText('干涉区域', interferenceX, interferenceY + 20);
            

            ctx.fillStyle = isWhiteLight ? 'white' : 'black';
            ctx.fillText('薄膜', (filmLeft + filmRight)/2, centerY);
            

            drawAngleMarker(ctx, incidentX, incidentY, incidenceAngle, 30, '入射角');
            drawAngleMarker(ctx, incidentX, incidentY, refractionAngle, 50, '折射角');
        }

        function drawColoredRay(ctx, x1, y1, x2, y2, color) {
            if (typeof color === 'string' && color.includes('gradient')) {
                const angle = Math.atan2(y2 - y1, x2 - x1);
                const gradient = ctx.createLinearGradient(x1, y1, x2, y2);
                
                if (color.includes('#9400d3')) {
           
                    gradient.addColorStop(0, '#9400d3');
                    gradient.addColorStop(1, '#0000ff');
                } else if (color.includes('#adff2f')) {
              
                    gradient.addColorStop(0, '#adff2f');
                    gradient.addColorStop(1, '#ffff00');
                } else if (color.includes('#ff00ff')) {
               
                    gradient.addColorStop(0, '#ff00ff');
                    gradient.addColorStop(1, '#ff1493');
                } else if (color.includes('#ff4500')) {
              
                    gradient.addColorStop(0, '#ff4500');
                    gradient.addColorStop(1, '#ff0000');
                } else if (color.includes('#00ffff')) {
               
                    gradient.addColorStop(0, '#00ffff');
                    gradient.addColorStop(1, '#00ced1');
                }
                
                ctx.strokeStyle = gradient;
            } else {
          
                ctx.strokeStyle = color;
            }
            
            ctx.lineWidth = 3;
            ctx.beginPath();
            ctx.moveTo(x1, y1);
            ctx.lineTo(x2, y2);
            ctx.stroke();
            

            const angle = Math.atan2(y2 - y1, x2 - x1);
            const headLength = 12;
            
            ctx.beginPath();
            ctx.moveTo(x2, y2);
            ctx.lineTo(
                x2 - headLength * Math.cos(angle - Math.PI / 6),
                y2 - headLength * Math.sin(angle - Math.PI / 6)
            );
            ctx.moveTo(x2, y2);
            ctx.lineTo(
                x2 - headLength * Math.cos(angle + Math.PI / 6),
                y2 - headLength * Math.sin(angle + Math.PI / 6)
            );
            ctx.stroke();
            
            const glowGradient = ctx.createRadialGradient(
                x2, y2, 0,
                x2, y2, 20
            );
            glowGradient.addColorStop(0, 'rgba(255, 255, 255, 0.8)');
            glowGradient.addColorStop(1, 'rgba(255, 255, 255, 0)');
            
            ctx.fillStyle = glowGradient;
            ctx.beginPath();
            ctx.arc(x2, y2, 20, 0, Math.PI * 2);
            ctx.fill();
        }
        
        function drawLightRay(ctx, x1, y1, x2, y2, color) {

            ctx.strokeStyle = color;
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(x1, y1);
            ctx.lineTo(x2, y2);
            ctx.stroke();
            
            const angle = Math.atan2(y2 - y1, x2 - x1);
            const headLength = 10;
            
            ctx.beginPath();
            ctx.moveTo(x2, y2);
            ctx.lineTo(
                x2 - headLength * Math.cos(angle - Math.PI / 6),
                y2 - headLength * Math.sin(angle - Math.PI / 6)
            );
            ctx.moveTo(x2, y2);
            ctx.lineTo(
                x2 - headLength * Math.cos(angle + Math.PI / 6),
                y2 - headLength * Math.sin(angle + Math.PI / 6)
            );
            ctx.stroke();
            

            const gradient = ctx.createRadialGradient(
                x2, y2, 0,
                x2, y2, 15
            );
            gradient.addColorStop(0, color);
            gradient.addColorStop(1, 'rgba(255, 255, 255, 0)');
            
            ctx.fillStyle = gradient;
            ctx.beginPath();
            ctx.arc(x2, y2, 15, 0, Math.PI * 2);
            ctx.fill();
        }
        
  
        function drawAngleMarker(ctx, x, y, angle, radius, label) {
            ctx.strokeStyle = isWhiteLight ? 'rgba(255, 255, 255, 0.7)' : 'rgba(0, 0, 0, 0.7)';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.arc(x, y, radius, 0, angle);
            ctx.stroke();
            

            ctx.font = '12px Arial';
            ctx.fillStyle = isWhiteLight ? 'white' : 'black';
            ctx.fillText(
                label + ': ' + (angle * 180 / Math.PI).toFixed(1) + '°',
                x + radius * Math.cos(angle / 2),
                y + radius * Math.sin(angle / 2)
            );
        }
        

        function drawInterferencePatterns() {
            const wavelength = parseFloat(wavelengthSlider.value);
            const thickness = parseFloat(thicknessSlider.value);
            const refractiveIndex = parseFloat(refractiveIndexSlider.value);
            const incidenceAngle = parseFloat(incidenceAngleSlider.value) * Math.PI / 180;
            

            const refractionAngle = Math.asin(Math.sin(incidenceAngle) / refractiveIndex);
            

            const opticalPathDiff = 2 * refractiveIndex * thickness * Math.cos(refractionAngle);
            

            let phaseDiff = (2 * Math.PI * opticalPathDiff) / wavelength;
            

            drawInterferencePattern(upperSurfaceCanvas, phaseDiff, true, thickness);
            
           
            drawInterferencePattern(lowerSurfaceCanvas, phaseDiff + Math.PI, false, thickness);
        }
        

        function drawInterferencePattern(canvas, phaseDiff, isUpper, thickness) {
            const ctx = canvas.getContext('2d');
            const width = canvas.width;
            const height = canvas.height;
            
            ctx.clearRect(0, 0, width, height);
            
            if (isWhiteLight) {
       
                let prevThickness = lastThickness;
                let currentThickness = thickness;
                
          
                if (Math.abs(prevThickness - currentThickness) > 50) {
              
                    const transitionFactor = Math.min(1, (currentThickness - prevThickness) / 100);
                    

                    const tempCanvas = document.createElement('canvas');
                    tempCanvas.width = width;
                    tempCanvas.height = height;
                    const tempCtx = tempCanvas.getContext('2d');
                    
   
                    tempCtx.globalAlpha = 0.5;
                    drawWhiteLightPattern(tempCtx, width, height, prevThickness, isUpper);
                    

                    tempCtx.globalAlpha = 0.5;
                    drawWhiteLightPattern(tempCtx, width, height, currentThickness, isUpper);
                    
        
                    ctx.drawImage(tempCanvas, 0, 0);
                } else {
                    
                    drawWhiteLightPattern(ctx, width, height, thickness, isUpper);
                }
            } else {
               
                drawMonochromaticPattern(ctx, width, height, phaseDiff, isUpper);
            }
        }
                function drawWhiteLightPattern(ctx, width, height, thickness, isUpper) {
            if (thickness < 50) {
    
                ctx.fillStyle = isUpper ? '#000000' : '#ffffff';
                ctx.fillRect(0, 0, width, height);
            } else if (thickness < 150) {
                
                if (isUpper) {
                    const gradient = ctx.createLinearGradient(0, 0, width, 0);
                    gradient.addColorStop(0, '#9400d3');
                    gradient.addColorStop(1, '#0000ff');
                    ctx.fillStyle = gradient;
                } else {
                    const gradient = ctx.createLinearGradient(0, 0, width, 0);
                    gradient.addColorStop(0, '#adff2f');
                    gradient.addColorStop(1, '#ffff00');
                    ctx.fillStyle = gradient;
                }
                ctx.fillRect(0, 0, width, height);
            } else if (thickness < 250) {

                if (isUpper) {
                    ctx.fillStyle = '#00ff00';
                } else {
                    const gradient = ctx.createLinearGradient(0, 0, width, 0);
                    gradient.addColorStop(0, '#ff00ff');
                    gradient.addColorStop(1, '#ff1493');
                    ctx.fillStyle = gradient;
                }
                ctx.fillRect(0, 0, width, height);
            } else {
   
                if (isUpper) {
                    const gradient = ctx.createLinearGradient(0, 0, width, 0);
                    gradient.addColorStop(0, '#ff4500');
                    gradient.addColorStop(1, '#ff0000');
                    ctx.fillStyle = gradient;
                } else {
                    const gradient = ctx.createLinearGradient(0, 0, width, 0);
                    gradient.addColorStop(0, '#00ffff');
                    gradient.addColorStop(1, '#00ced1');
                    ctx.fillStyle = gradient;
                }
                ctx.fillRect(0, 0, width, height);
            }
            
           
            ctx.globalCompositeOperation = 'overlay';
            ctx.fillStyle = 'rgba(255, 255, 255, 0.1)';
            
       
            const stripeCount = 8 + Math.floor(thickness / 100);
            for (let i = 0; i < stripeCount; i++) {
                const x = i * width / stripeCount;
                const radius = height / 3 + Math.sin(Date.now() / 1000 + i) * height / 20;
                
                ctx.beginPath();
                ctx.arc(x, height/2, radius, 0, Math.PI * 2);
                ctx.fill();
            }
            ctx.globalCompositeOperation = 'source-over';
        }
        
        
        function drawMonochromaticPattern(ctx, width, height, phaseDiff, isUpper) {
            const intensity = 0.5 * (1 + Math.cos(phaseDiff));
            let r, g, b;
            
            if (isUpper) {
                
                r = Math.floor(currentColor[0] * intensity);
                g = Math.floor(currentColor[1] * intensity);
                b = Math.floor(currentColor[2] * intensity);
            } else {
                
                r = Math.floor(currentColor[0] * intensity);
                g = Math.floor(currentColor[1] * intensity);
                b = Math.floor(currentColor[2] * intensity);
            }
            
            ctx.fillStyle = `rgb(${r},${g},${b})`;
            ctx.fillRect(0, 0, width, height);
            

            const stripeCount = 10;
            for (let i = 0; i < stripeCount; i++) {
                const pos = i / stripeCount * width;
                const stripeWidth = width / (stripeCount * 2);
                

                const offset = Math.sin(Date.now() / 1000) * stripeWidth / 4;
                
                const gradient = ctx.createLinearGradient(pos + offset, 0, pos + stripeWidth + offset, 0);
                gradient.addColorStop(0, `rgba(${r},${g},${b},0)`);
                gradient.addColorStop(0.5, `rgba(${r},${g},${b},${intensity})`);
                gradient.addColorStop(1, `rgba(${r},${g},${b},0)`);
                
                ctx.fillStyle = gradient;
                ctx.fillRect(pos, 0, stripeWidth, height);
            }
        }
    </script>
</body>
</html>
