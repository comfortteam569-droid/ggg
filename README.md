```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>말미잘 자포 방출 정밀 시각화</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;700;900&display=swap');
        
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: #f1f5f9;
        }

        /* 부드러운 애니메이션을 위한 트랜지션 */
        .fade-in {
            animation: fadeIn 1s ease-in forwards;
        }

        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }

        .membrane-layer {
            stroke: #64748b;
            stroke-width: 4;
            stroke-dasharray: 8 4;
        }

        .stage-card {
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
        }

        .active-step {
            border-color: #2563eb;
            background-color: #eff6ff;
            box-shadow: 0 4px 20px -2px rgba(37, 99, 235, 0.2);
        }

        /* 관 전개 애니메이션 속도 늦춤 */
        .eversion-slow {
            stroke-dasharray: 1000;
            stroke-dashoffset: 1000;
            animation: drawLong 4s forwards ease-in-out;
        }

        @keyframes drawLong {
            to { stroke-dashoffset: 0; }
        }

        /* 이온 흐름 속도 조절 */
        .ion-slow-move {
            animation: ionMove 3s infinite linear;
        }

        @keyframes ionMove {
            from { transform: translateY(0); opacity: 1; }
            to { transform: translateY(80px); opacity: 0; }
        }
    </style>
</head>
<body class="min-h-screen p-4 md:p-8">
    <div class="max-w-6xl mx-auto">
        <header class="text-center mb-10">
            <h1 class="text-3xl md:text-4xl font-black text-slate-900 mb-2">자포세포 반응성 및 방출 메커니즘</h1>
            <p class="text-slate-500 font-medium tracking-tight text-sm md:text-base">화학적 조절과 기계적 자극의 통합적 메커니즘 시각화</p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
            <!-- 시각화 캔버스 -->
            <div class="lg:col-span-8 space-y-6">
                <div class="relative bg-white rounded-[2rem] shadow-xl border border-slate-200 overflow-hidden aspect-[16/10] flex items-center justify-center">
                    <svg id="main-canvas" viewBox="0 0 800 500" class="w-full h-full">
                        <defs>
                            <!-- 세포막 질감 -->
                            <pattern id="membranePattern" x="0" y="0" width="20" height="20" patternUnits="userSpaceOnUse">
                                <circle cx="10" cy="5" r="3" fill="#cbd5e1"/>
                                <line x1="10" y1="8" x2="10" y2="15" stroke="#cbd5e1" stroke-width="1"/>
                            </pattern>
                            <!-- 캡슐 그래디언트 -->
                            <radialGradient id="capsuleFill" cx="50%" cy="50%" r="50%">
                                <stop offset="0%" stop-color="#f0f9ff"/>
                                <stop offset="100%" stop-color="#bae6fd"/>
                            </radialGradient>
                        </defs>

                        <!-- 배경 레이어 (세포 환경) -->
                        <rect x="0" y="0" width="800" height="200" fill="#f8fafc"/> <!-- 외부 -->
                        <rect x="0" y="230" width="800" height="270" fill="#ffffff"/> <!-- 내부 -->
                        
                        <!-- 세포막 -->
                        <rect x="0" y="200" width="800" height="30" fill="url(#membranePattern)" />
                        <line x1="0" y1="200" x2="800" y2="200" stroke="#94a3b8" stroke-width="2"/>
                        <line x1="0" y1="230" x2="800" y2="230" stroke="#94a3b8" stroke-width="2"/>

                        <!-- 가변 콘텐츠 영역 -->
                        <g id="content-layer"></g>
                    </svg>

                    <!-- 단계 안내 레이블 -->
                    <div id="step-label" class="absolute top-6 left-6 px-5 py-2 bg-blue-600 text-white rounded-2xl text-sm font-black shadow-lg">
                        단계 ㄱ
                    </div>
                </div>

                <!-- 텍스트 설명 카드 -->
                <div class="bg-white p-8 rounded-[2rem] shadow-sm border border-slate-200 min-h-[160px]">
                    <h2 id="step-title" class="text-2xl font-bold text-slate-800 mb-3 underline decoration-blue-200 decoration-8 underline-offset-1">제목</h2>
                    <p id="step-desc" class="text-slate-600 text-lg leading-relaxed"></p>
                </div>
            </div>

            <!-- 우측 리스트 및 컨트롤 -->
            <div class="lg:col-span-4 flex flex-col gap-4">
                <div id="nav-list" class="space-y-3">
                    <!-- JS에서 동적 생성됨 -->
                </div>

                <div class="mt-4 p-6 bg-slate-900 rounded-[2rem] text-white">
                    <h3 class="text-xs font-bold text-slate-400 uppercase tracking-widest mb-4">Playback Control</h3>
                    <div class="flex flex-col gap-3">
                        <button id="toggle-play" class="w-full py-4 bg-blue-500 hover:bg-blue-600 rounded-2xl font-bold transition-all flex items-center justify-center gap-2">
                            <span id="play-status-icon">▶</span> <span id="play-status-text">자동 재생 시작</span>
                        </button>
                        <div class="flex gap-2">
                            <button onclick="moveStep(-1)" class="flex-1 py-3 bg-slate-800 hover:bg-slate-700 rounded-xl transition-colors">이전</button>
                            <button onclick="moveStep(1)" class="flex-1 py-3 bg-slate-800 hover:bg-slate-700 rounded-xl transition-colors">다음</button>
                        </div>
                    </div>
                    <div class="mt-4 bg-slate-800 h-1 rounded-full overflow-hidden">
                        <div id="progress-indicator" class="bg-blue-400 h-full w-0 transition-all duration-300"></div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        const steps = [
            {
                label: "ㄱ",
                title: "화학적 자극: 반응성 리셋",
                desc: "아세틸콜린(ACh)이 수용체에 결합하여 세포막을 과분극시킵니다. 이 과정이 선행되어야만 비로소 '잠겨있던' nCaV 채널이 반응할 수 있는 준비 상태가 됩니다.",
                render: () => `
                    <g class="fade-in">
                        <!-- ACh 수용체 -->
                        <path d="M 280 200 L 320 200 L 310 180 L 290 180 Z" fill="#3b82f6" stroke="#1d4ed8" stroke-width="2"/>
                        <!-- ACh 이온들 (천천히 내려옴) -->
                        <circle cx="300" cy="50" r="6" fill="#60a5fa">
                            <animate attributeName="cy" from="50" to="185" dur="3s" fill="freeze" />
                        </circle>
                        <text x="315" y="60" class="text-[12px] fill-blue-500 font-bold italic">ACh (신경전달물질)</text>

                        <!-- K+ 유출 (과분극 묘사) -->
                        <g transform="translate(300, 230)">
                            <circle cx="0" cy="10" r="4" fill="#10b981" class="ion-slow-move" />
                            <text x="15" y="30" class="text-[12px] fill-emerald-600 font-bold">K⁺ 유출 (과분극 중...)</text>
                        </g>

                        <!-- nCaV 채널 (불활성 상태에서 대기로) -->
                        <rect x="480" y="195" width="40" height="40" rx="4" fill="#ef4444" opacity="0.3" stroke="#ef4444" stroke-width="2" />
                        <text x="470" y="180" class="text-[10px] fill-red-400 font-bold italic">nCaV (잠금 해제 중)</text>
                    </g>
                `
            },
            {
                label: "ㄴ",
                title: "기계적 자극: 결합적 활성화",
                desc: "ㄱ단계에서 준비된 nCaV는 이제 기계적 접촉에 반응합니다. 화학적 배경과 물리적 자극이 동시에 충족될 때 채널이 열리고 대량의 Ca²⁺가 유입됩니다.",
                render: () => `
                    <g class="fade-in">
                        <!-- 준비된 nCaV 채널 (열림) -->
                        <g transform="translate(400, 215)">
                            <rect x="-25" y="-15" width="20" height="30" fill="#10b981" />
                            <rect x="5" y="-15" width="20" height="30" fill="#10b981" />
                        </g>
                        
                        <!-- 기계적 자극 (먹이와의 접촉) - 아주 천천히 접근 -->
                        <path d="M 400 -50 L 400 120" stroke="#475569" stroke-width="15" stroke-linecap="round">
                            <animate attributeName="d" values="M 400 -50 L 400 100; M 400 -50 L 400 185; M 400 -50 L 400 100" dur="4s" repeatCount="indefinite" />
                        </path>
                        <text x="420" y="50" class="text-[16px] fill-slate-800 font-black">먹이와의 접촉 (기계적 자극)</text>

                        <!-- Ca2+ 대량 유입 (천천히 이동) -->
                        <g transform="translate(400, 150)">
                            <circle cx="0" cy="0" r="10" fill="#f59e0b" filter="url(#glow)">
                                <animate attributeName="cy" from="0" to="120" dur="2s" repeatCount="indefinite" />
                            </circle>
                        </g>
                        <text x="430" y="270" class="text-[14px] fill-amber-600 font-black">Ca²⁺ 유입 (방출 결정 신호)</text>
                        <text x="250" y="100" class="text-[12px] fill-blue-600 font-bold underline">ACh 조건 충족됨 ✓</text>
                    </g>
                `
            },
            {
                label: "ㄷ",
                title: "압력 형성: 삼투압의 힘",
                desc: "유입된 칼슘 신호로 방출이 개시됩니다. 캡슐 내부의 고분자 기질(P-γ-G)이 삼투압을 발생시키며, 외부의 물이 캡슐 안으로 거세게 유입되어 에너지를 형성합니다.",
                render: () => `
                    <g class="fade-in">
                        <!-- 자포 캡슐 (점진적 팽창) -->
                        <circle cx="400" cy="360" r="60" fill="url(#capsuleFill)" stroke="#0ea5e9" stroke-width="4">
                            <animate attributeName="r" values="60;90;85" dur="4s" repeatCount="indefinite" />
                        </circle>
                        
                        <!-- 내부 꼬인 Tubule 상징 -->
                        <path d="M 370 360 Q 400 390 430 360 Q 400 330 370 360" fill="none" stroke="#ef4444" stroke-width="3" opacity="0.4">
                            <animateTransform attributeName="transform" type="rotate" from="0 400 360" to="360 400 360" dur="8s" repeatCount="indefinite" />
                        </path>

                        <!-- 물 유입 화살표 (천천히 반복) -->
                        <g>
                            ${[0, 90, 180, 270].map(angle => `
                                <path d="M ${400 + Math.cos(angle)*130} ${360 + Math.sin(angle)*130} L ${400 + Math.cos(angle)*100} ${360 + Math.sin(angle)*100}" 
                                      stroke="#3b82f6" stroke-width="3" opacity="0">
                                    <animate attributeName="opacity" values="0;1;0" dur="2s" repeatCount="indefinite" />
                                </path>
                            `).join('')}
                        </g>
                        <text x="400" y="365" text-anchor="middle" class="text-[14px] fill-blue-700 font-bold">삼투적 팽윤 (H₂O 유입)</text>
                    </g>
                `
            },
            {
                label: "ㄹ",
                title: "에너지 저장: 탄성 한계",
                desc: "캡슐벽의 미니콜라겐 구조가 한계까지 늘어납니다. 이 탄성 에너지는 고무줄을 끝까지 당긴 것과 같으며, 나중에 자포관을 밀어내는 핵심 동력이 됩니다.",
                render: () => `
                    <g class="fade-in">
                        <!-- 최대 팽창 캡슐 -->
                        <circle cx="400" cy="360" r="110" fill="#f0f9ff" stroke="#1d4ed8" stroke-width="10" stroke-dasharray="15 5">
                            <animate attributeName="stroke-width" values="10;14;10" dur="1s" repeatCount="indefinite" />
                        </circle>
                        
                        <text x="400" y="365" text-anchor="middle" class="text-[20px] fill-blue-900 font-black">탄성 에너지 저장됨</text>
                        <text x="520" y="480" class="text-[12px] fill-slate-400 font-bold">복합 단백질벽 (Mini-collagen)</text>
                    </g>
                `
            },
            {
                label: "ㅁ",
                title: "단계적 방출: Shaft & Tubule",
                desc: "1. Shaft가 먼저 튀어나와 표적을 찌릅니다. 2. 뒤집힘(Eversion)이 일어나며 통로가 확보됩니다. 3. 긴 관(Tubule)이 그 안쪽에서 바깥쪽으로 계속해서 뒤집히며 전개됩니다.",
                render: () => `
                    <g class="fade-in">
                        <!-- 캡슐 하단 고정 -->
                        <path d="M 350 480 Q 400 500 450 480 L 430 400 L 370 400 Z" fill="#bae6fd" stroke="#0369a1" stroke-width="3" />
                        
                        <!-- 방출 애니메이션 (속도 매우 늦춤) -->
                        <g transform="translate(400, 400)">
                            <!-- Shaft (빠르게 튀어나오지만 뒤집힘은 천천히) -->
                            <rect x="-15" y="-70" width="30" height="70" fill="#60a5fa" stroke="#1d4ed8" stroke-width="2">
                                <animate attributeName="height" values="0;70" dur="1s" fill="freeze" />
                                <animate attributeName="y" values="0;-70" dur="1s" fill="freeze" />
                            </rect>
                            
                            <!-- Tubule (매우 천천히 전개됨) -->
                            <path d="M 0 -70 L 0 -350" stroke="#ef4444" stroke-width="6" stroke-linecap="round" class="eversion-slow" />
                            
                            <!-- 가시(Barbs) 전개 -->
                            <path d="M -15 -50 L -30 -60 M 15 -50 L 30 -60" stroke="#1d4ed8" stroke-width="3">
                                <animate attributeName="opacity" from="0" to="1" dur="2s" fill="freeze" />
                            </path>
                        </g>

                        <text x="450" y="100" class="text-[16px] fill-red-600 font-black italic">Tubule Eversion 전개 중...</text>
                        <text x="300" y="320" class="text-[14px] fill-blue-700 font-bold">Shaft Discharge</text>
                    </g>
                `
            }
        ];

        let currentStep = 0;
        let isAutoPlaying = false;
        let autoPlayTimer = null;
        const AUTO_PLAY_INTERVAL = 7000; // 7초로 늦춤 (설명 읽고 애니메이션 볼 시간 확보)

        function updateUI() {
            const step = steps[currentStep];
            document.getElementById('content-layer').innerHTML = step.render();
            document.getElementById('step-title').innerText = step.title;
            document.getElementById('step-desc').innerText = step.desc;
            document.getElementById('step-label').innerText = `단계 ${step.label}`;
            document.getElementById('progress-indicator').style.width = `${((currentStep + 1) / steps.length) * 100}%`;
            
            // 네비게이션 리스트 갱신
            document.querySelectorAll('.stage-card').forEach((card, idx) => {
                if (idx === currentStep) card.classList.add('active-step');
                else card.classList.remove('active-step');
            });
        }

        function buildNav() {
            const nav = document.getElementById('nav-list');
            nav.innerHTML = '';
            steps.forEach((step, idx) => {
                const card = document.createElement('div');
                card.className = `stage-card p-4 rounded-2xl border-2 flex items-center gap-4 bg-white cursor-pointer`;
                card.innerHTML = `
                    <div class="w-10 h-10 rounded-full bg-slate-900 text-white flex items-center justify-center font-black">${step.label}</div>
                    <div class="font-bold text-slate-800 text-sm">${step.title}</div>
                `;
                card.onclick = () => {
                    if (isAutoPlaying) toggleAutoPlay();
                    currentStep = idx;
                    updateUI();
                };
                nav.appendChild(card);
            });
        }

        function moveStep(dir) {
            if (isAutoPlaying) toggleAutoPlay();
            currentStep = (currentStep + dir + steps.length) % steps.length;
            updateUI();
        }

        function toggleAutoPlay() {
            isAutoPlaying = !isAutoPlaying;
            const btn = document.getElementById('toggle-play');
            const icon = document.getElementById('play-status-icon');
            const text = document.getElementById('play-status-text');

            if (isAutoPlaying) {
                btn.classList.replace('bg-blue-500', 'bg-red-500');
                icon.innerText = '■';
                text.innerText = '자동 재생 정지';
                autoPlayTimer = setInterval(() => {
                    currentStep = (currentStep + 1) % steps.length;
                    updateUI();
                }, AUTO_PLAY_INTERVAL);
            } else {
                btn.classList.replace('bg-red-500', 'bg-blue-500');
                icon.innerText = '▶';
                text.innerText = '자동 재생 시작';
                clearInterval(autoPlayTimer);
            }
        }

        document.getElementById('toggle-play').onclick = toggleAutoPlay;

        // 초기화
        buildNav();
        updateUI();
    </script>
</body>
</html>

```
