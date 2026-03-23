# Git-vollyeyanalisys
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>V-Master 프로배구 전력분석 시스템</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700;900&display=swap');
        body { font-family: 'Noto Sans KR', sans-serif; background-color: #0f172a; color: white; }
        .glass-panel { background: rgba(30, 41, 59, 0.7); backdrop-filter: blur(10px); border: 1px solid rgba(255,255,255,0.1); }
        .tab-btn.active { border-bottom: 4px solid #3b82f6; color: #60a5fa; font-weight: 900; }
        .blink-red { animation: blinker 1.5s linear infinite; }
        @keyframes blinker { 50% { opacity: 0; } }
        /* 스크롤바 숨기기 */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-thumb { background: #475569; rounded: 10px; }
    </style>
</head>
<body class="h-screen flex flex-col overflow-hidden">

    <!-- Top Navigation -->
    <header class="bg-gray-900 border-b border-gray-800 p-4 flex justify-between items-center shadow-lg shrink-0">
        <div class="flex items-center space-x-3">
            <div class="bg-blue-600 p-2 rounded-lg"><i data-lucide="activity" class="w-6 h-6 text-white"></i></div>
            <h1 class="text-2xl font-black tracking-wider text-white">V-MASTER <span class="text-sm font-normal text-blue-400 ml-2">통합 우승 전력분석 AI</span></h1>
        </div>
        <div class="flex space-x-2">
            <button onclick="switchTab('pre')" id="tab-pre" class="tab-btn active px-4 py-2 text-gray-400 hover:text-white transition">1. 경기 전 (선수관리)</button>
            <button onclick="switchTab('in')" id="tab-in" class="tab-btn px-4 py-2 text-gray-400 hover:text-white transition">2. 경기 중 (실시간 작전)</button>
            <button onclick="switchTab('post')" id="tab-post" class="tab-btn px-4 py-2 text-gray-400 hover:text-white transition">3. 경기 후 (2분 평가서)</button>
        </div>
        <div class="flex items-center space-x-4">
            <span class="text-sm text-gray-400 flex items-center"><i data-lucide="clock" class="w-4 h-4 mr-1"></i> <span id="clock">19:00:00</span></span>
            <div class="w-10 h-10 bg-gray-700 rounded-full flex items-center justify-center border-2 border-blue-500"><i data-lucide="user" class="w-6 h-6"></i></div>
        </div>
    </header>

    <!-- Main Content Area -->
    <main class="flex-1 overflow-y-auto p-6">
        
        <!-- ==========================================
             1. 경기 전 (평소/선수단 관리 및 라인업 추천)
             ========================================== -->
        <section id="view-pre" class="block space-y-6 max-w-7xl mx-auto">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                
                <!-- 선수 자가 진단 입력창 -->
                <div class="glass-panel rounded-2xl p-6 lg:col-span-1 shadow-xl">
                    <h2 class="text-xl font-bold mb-4 flex items-center text-blue-400"><i data-lucide="clipboard-edit" class="w-5 h-5 mr-2"></i> 일일 선수 자가진단 (모바일)</h2>
                    <form class="space-y-4" onsubmit="event.preventDefault(); alert('데이터가 AI 서버로 전송되었습니다.');">
                        <div class="grid grid-cols-2 gap-4">
                            <div><label class="text-xs text-gray-400">이름</label><input type="text" value="김배구" class="w-full bg-gray-800 rounded p-2 text-white border border-gray-700" readonly></div>
                            <div><label class="text-xs text-gray-400">포지션</label><input type="text" value="레프트 (OH)" class="w-full bg-gray-800 rounded p-2 text-white border border-gray-700" readonly></div>
                        </div>
                        <div>
                            <label class="text-xs text-gray-400">오늘의 정신 컨디션 및 스트레스</label>
                            <input type="range" min="1" max="10" value="8" class="w-full mt-2 accent-blue-500">
                            <div class="flex justify-between text-xs text-gray-500 mt-1"><span>나쁨</span><span>최상</span></div>
                        </div>
                        <div>
                            <label class="text-xs text-gray-400">수면 시간 및 질</label>
                            <select class="w-full bg-gray-800 rounded p-2 text-white border border-gray-700 mt-1">
                                <option>8시간 이상 (매우 좋음)</option><option>6~8시간 (보통)</option><option>6시간 미만 (피곤함)</option>
                            </select>
                        </div>
                        <div>
                            <label class="text-xs text-gray-400">통증 및 불편 사항 (직접 입력)</label>
                            <textarea class="w-full bg-gray-800 rounded p-2 text-white border border-gray-700 mt-1" rows="2" placeholder="예: 오른쪽 무릎 약간 뻐근함"></textarea>
                        </div>
                        <button class="w-full bg-blue-600 hover:bg-blue-500 text-white font-bold py-3 rounded-lg transition">데이터 전송</button>
                    </form>
                </div>

                <!-- 코칭스태프 평가 & AI 스타팅 추천 -->
                <div class="glass-panel rounded-2xl p-6 lg:col-span-2 shadow-xl flex flex-col">
                    <div class="flex justify-between items-center mb-6">
                        <h2 class="text-xl font-bold flex items-center text-yellow-400"><i data-lucide="cpu" class="w-5 h-5 mr-2"></i> AI 최적 스타팅 6인 추천</h2>
                        <span class="bg-gray-800 px-3 py-1 rounded text-xs text-gray-400 border border-gray-700">상대팀: 챔피언스 (전력분석 반영됨)</span>
                    </div>
                    
                    <div class="bg-gray-800/50 p-4 rounded-xl border border-gray-700 mb-6">
                        <p class="text-sm text-gray-300 leading-relaxed">
                            <strong class="text-blue-400">💡 보좌관 AI 분석:</strong> 상대팀 주포의 공격 방향이 레프트에 60% 집중됩니다. 오늘 아침 <strong>[블로킹 감각 95점]</strong>을 기록하고 숙소 면담에서 컨디션이 최상으로 평가된 김철수(MB) 선수를 선발 출전시켜 상대 공격을 차단하는 것을 적극 건의합니다. 용병 선수의 무릎 미세 통증이 보고되었으므로 백업 라이트를 즉각 대기시켜야 합니다.
                        </p>
                    </div>

                    <!-- 코트 그래픽 라인업 -->
                    <div class="flex-1 bg-green-900/20 border-2 border-green-500/30 rounded-xl relative overflow-hidden flex items-center justify-center p-4">
                        <!-- 코트 라인 -->
                        <div class="absolute inset-4 border-2 border-white/20"></div>
                        <div class="absolute inset-x-4 top-1/3 border-b-2 border-white/20"></div>
                        
                        <!-- 선수 배치 -->
                        <div class="grid grid-cols-3 gap-x-8 gap-y-12 w-full max-w-md z-10 relative">
                            <!-- 전위 -->
                            <div class="flex flex-col items-center">
                                <div class="w-12 h-12 rounded-full bg-blue-600 flex items-center justify-center font-bold border-2 border-green-400 shadow-[0_0_15px_rgba(74,222,128,0.5)]">OH</div>
                                <span class="text-xs mt-2">김배구 (🟢95%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-12 h-12 rounded-full bg-blue-600 flex items-center justify-center font-bold border-2 border-green-400 shadow-[0_0_15px_rgba(74,222,128,0.5)]">MB</div>
                                <span class="text-xs mt-2 text-yellow-300 font-bold">김철수 (🟢98%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-12 h-12 rounded-full bg-blue-600 flex items-center justify-center font-bold border-2 border-yellow-400">OP</div>
                                <span class="text-xs mt-2 text-red-300">용병A (🟡75%)</span>
                            </div>
                            <!-- 후위 -->
                            <div class="flex flex-col items-center">
                                <div class="w-12 h-12 rounded-full bg-blue-800 flex items-center justify-center font-bold border-2 border-green-400">OH</div>
                                <span class="text-xs mt-2">이수비 (🟢90%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-12 h-12 rounded-full bg-gray-600 flex items-center justify-center font-bold border-2 border-green-400">L</div>
                                <span class="text-xs mt-2">박디그 (🟢88%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-12 h-12 rounded-full bg-blue-800 flex items-center justify-center font-bold border-2 border-green-400">S</div>
                                <span class="text-xs mt-2">최세터 (🟢92%)</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- ==========================================
             2. 경기 중 (실시간 동영상 분석 및 위기 대응)
             ========================================== -->
        <section id="view-in" class="hidden space-y-6 h-full max-w-screen-2xl mx-auto">
            <div class="grid grid-cols-1 lg:grid-cols-4 gap-6 h-[80vh]">
                
                <!-- 비디오 분석 및 코트 맵 (좌측 3칸) -->
                <div class="lg:col-span-3 flex flex-col space-y-4">
                    <!-- AI 카메라 영상 송출창 -->
                    <div class="bg-black rounded-2xl border border-gray-700 h-2/3 relative overflow-hidden flex items-center justify-center shadow-2xl">
                        <img src="https://images.unsplash.com/photo-1612872087720-bb876e2e67d1?ixlib=rb-4.0.3&auto=format&fit=crop&w=1200&q=80" class="absolute inset-0 w-full h-full object-cover opacity-40">
                        <!-- AI Vision Tagging 그래픽 요소 -->
                        <div class="absolute inset-0 bg-[url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0MCIgaGVpZ2h0PSI0MCI+PHBhdGggZD0iTTAgMGg0MHY0MEgweiIgZmlsbD0ibm9uZSIvPjxwYXRoIGQ0iTTAgMGgxdjQwSDB6TTAgMGg0MHYxSDB6IiBmaWxsPSJyZ2JhKDI1NSwyNTUsMjU1LDAuMSkiLz48L3N2Zz4=')]"></div>
                        <div class="absolute top-1/4 right-1/4 border-2 border-red-500 w-32 h-40 rounded flex flex-col justify-end p-1">
                            <span class="bg-red-500 text-white text-[10px] px-1 w-max mb-1">상대 용병 (타점: 310cm)</span>
                            <span class="bg-yellow-500 text-white text-[10px] px-1 w-max">스파이크 속도: 110km/h</span>
                        </div>
                        <!-- 상단 상태 바 -->
                        <div class="absolute top-4 left-4 bg-red-600 text-white px-3 py-1 rounded text-sm font-bold animate-pulse flex items-center">
                            <i data-lucide="radio" class="w-4 h-4 mr-2"></i> LIVE AI Tracking (자체 카메라 연동)
                        </div>
                    </div>

                    <!-- 실시간 컨트롤 패널 (리듬 유지의 핵심) -->
                    <div class="glass-panel rounded-2xl h-1/3 p-4 flex flex-col">
                        <h3 class="text-sm font-bold text-gray-400 mb-2">실시간 데이터 태깅 (분석관 터치 입력)</h3>
                        <div class="flex-1 grid grid-cols-4 gap-3">
                            <button onclick="logEvent('공격 성공', 'bg-blue-600')" class="bg-gray-800 hover:bg-blue-600 border border-gray-600 rounded-xl flex flex-col items-center justify-center transition active:scale-95">
                                <i data-lucide="crosshair" class="w-6 h-6 mb-1 text-blue-400"></i><span class="font-bold text-sm">공격 성공</span>
                            </button>
                            <button onclick="logEvent('블로킹 성공', 'bg-green-600')" class="bg-gray-800 hover:bg-green-600 border border-gray-600 rounded-xl flex flex-col items-center justify-center transition active:scale-95">
                                <i data-lucide="shield" class="w-6 h-6 mb-1 text-green-400"></i><span class="font-bold text-sm">블로킹 성공</span>
                            </button>
                            <!-- 경기 리듬을 깨는 치명적 요소 버튼 -->
                            <button onclick="logEvent('평범한 서브 (상대 찬스 허용)', 'bg-yellow-600'); dropRhythm(10);" class="bg-gray-800 hover:bg-yellow-600 border border-gray-600 rounded-xl flex flex-col items-center justify-center transition active:scale-95">
                                <i data-lucide="wind" class="w-6 h-6 mb-1 text-yellow-400"></i><span class="font-bold text-sm">평범한 서브</span>
                            </button>
                            <button onclick="logEvent('어이없는 범실 (리듬 붕괴)', 'bg-red-600'); dropRhythm(25);" class="bg-red-900/50 hover:bg-red-600 border border-red-500 rounded-xl flex flex-col items-center justify-center transition active:scale-95 shadow-[0_0_15px_rgba(239,68,68,0.3)]">
                                <i data-lucide="alert-octagon" class="w-6 h-6 mb-1 text-red-400"></i><span class="font-bold text-sm">어이없는 범실</span>
                            </button>
                        </div>
                    </div>
                </div>

                <!-- 실시간 AI 벤치 알람 및 상황판 (우측 1칸) -->
                <div class="glass-panel rounded-2xl p-4 lg:col-span-1 flex flex-col h-full overflow-hidden">
                    <h2 class="text-lg font-black mb-4 border-b border-gray-700 pb-2 text-white flex items-center">
                        <i data-lucide="zap" class="w-5 h-5 mr-2 text-yellow-400"></i> AI 벤치 작전 지시
                    </h2>
                    
                    <!-- 팀 모멘텀(리듬) 게이지 -->
                    <div class="mb-6">
                        <div class="flex justify-between text-xs font-bold mb-1">
                            <span class="text-gray-400">팀 리듬 (모멘텀) 지수</span>
                            <span id="rhythm-text" class="text-green-400">안정적 (85%)</span>
                        </div>
                        <div class="w-full bg-gray-800 rounded-full h-3 border border-gray-700 overflow-hidden">
                            <div id="rhythm-bar" class="bg-green-500 h-3 rounded-full transition-all duration-500" style="width: 85%"></div>
                        </div>
                    </div>

                    <!-- 타임아웃 경고 모달창 (숨김 상태) -->
                    <div id="timeout-alert" class="hidden bg-red-600 text-white p-4 rounded-xl mb-6 shadow-[0_0_20px_rgba(239,68,68,0.8)] blink-red border-2 border-white">
                        <div class="flex items-center justify-center mb-2"><i data-lucide="siren" class="w-8 h-8 mr-2"></i><span class="font-black text-xl">작전 타임 권장!</span></div>
                        <p class="text-xs text-center font-bold">어이없는 범실 누적. 흐름을 끊어야 합니다!</p>
                    </div>

                    <!-- 용병 교체 건의 카드 -->
                    <div class="bg-gray-800 border border-yellow-500/50 p-3 rounded-xl mb-6">
                        <div class="flex justify-between items-start mb-2">
                            <span class="text-xs font-bold text-yellow-500 border border-yellow-500 px-1 rounded">선수 교체 대기</span>
                            <i data-lucide="refresh-cw" class="w-4 h-4 text-yellow-500"></i>
                        </div>
                        <p class="text-sm font-bold mb-1">외인 용병 공격 성공률 28%</p>
                        <p class="text-xs text-gray-400">3연속 득점 실패. 백업 선발 준비 요망.</p>
                    </div>

                    <!-- 실시간 이벤트 로그 -->
                    <div class="flex-1 overflow-y-auto">
                        <h3 class="text-xs font-bold text-gray-500 mb-2">실시간 경기 로그</h3>
                        <ul id="event-log" class="space-y-2">
                            <li class="text-xs text-gray-300 bg-gray-800 p-2 rounded flex items-center border-l-2 border-blue-500"><span class="text-gray-500 w-10">1SET</span> 김배구 퀵오픈 성공</li>
                        </ul>
                    </div>
                </div>
            </div>
        </section>

        <!-- ==========================================
             3. 경기 후 (2분 요약 분석표 및 맞춤 스케줄)
             ========================================== -->
        <section id="view-post" class="hidden space-y-6 max-w-7xl mx-auto">
            <div class="text-center mb-8">
                <h2 class="text-3xl font-black text-white mb-2">경기 종합 분석 및 평가서 (Post-Match)</h2>
                <p class="text-gray-400">코칭스태프가 2분 안에 이해하고 내일 훈련에 즉시 적용할 수 있는 요약 대시보드입니다.</p>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- 5대 지표 레이더 차트 (시각화의 핵심) -->
                <div class="glass-panel rounded-2xl p-6 shadow-xl flex flex-col items-center justify-center">
                    <h3 class="text-lg font-bold mb-4 w-full text-left text-blue-400 flex items-center"><i data-lucide="pie-chart" class="w-5 h-5 mr-2"></i> 팀 5대 핵심 지표 분석</h3>
                    <div class="w-full max-w-[300px] relative">
                        <canvas id="radarChart"></canvas>
                    </div>
                    <div class="w-full mt-4 bg-gray-800/50 p-3 rounded-lg border border-red-500/30">
                        <p class="text-xs text-red-400 font-bold"><i data-lucide="alert-circle" class="w-3 h-3 inline"></i> 분석 결과: 수비 리시브 효율이 리그 평균(50%) 대비 현저히 낮음 (35%).</p>
                    </div>
                </div>

                <!-- 2분 요약 AI 평가서 -->
                <div class="glass-panel rounded-2xl p-6 lg:col-span-2 shadow-xl flex flex-col">
                    <h3 class="text-lg font-bold mb-4 text-yellow-400 flex items-center"><i data-lucide="file-text" class="w-5 h-5 mr-2"></i> 코칭스태프 2분 직관 평가서</h3>
                    
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 flex-1">
                        <!-- 강점 및 수훈 -->
                        <div class="bg-gray-800 border border-green-500/30 rounded-xl p-4">
                            <h4 class="font-bold text-green-400 mb-2 flex items-center"><i data-lucide="thumbs-up" class="w-4 h-4 mr-1"></i> 긍정적 요소 (Keep)</h4>
                            <ul class="text-sm text-gray-300 space-y-2 list-disc pl-4 marker:text-green-500">
                                <li><strong>중앙 속공 활용도 상승:</strong> 미들블로커 김철수의 속공 득점률 65% 달성 (상대 센터진 완벽 교란).</li>
                                <li><strong>클러치 상황 집중력:</strong> 20점 이후 점수대에서 범실이 단 1개로, 후반 멘탈 관리가 매우 우수했음.</li>
                            </ul>
                        </div>
                        
                        <!-- 치명적 약점 -->
                        <div class="bg-gray-800 border border-red-500/30 rounded-xl p-4">
                            <h4 class="font-bold text-red-400 mb-2 flex items-center"><i data-lucide="thumbs-down" class="w-4 h-4 mr-1"></i> 치명적 약점 (Fix)</h4>
                            <ul class="text-sm text-gray-300 space-y-2 list-disc pl-4 marker:text-red-500">
                                <li><strong>외인 용병 체력 저하 뚜렷:</strong> 3세트 이후 스파이크 타점이 12cm 하락하며 블로킹에 4회 연속 차단됨.</li>
                                <li><strong>플로터 서브 대처 미흡:</strong> 상대의 목적타(변화구) 서브에 레프트 라인의 리시브가 심각하게 흔들림.</li>
                            </ul>
                        </div>

                        <!-- 내일 훈련 처방전 (Action Plan) -->
                        <div class="bg-blue-900/30 border border-blue-500/50 rounded-xl p-4 md:col-span-2 mt-2">
                            <h4 class="font-bold text-blue-400 mb-3 flex items-center"><i data-lucide="calendar-check" class="w-5 h-5 mr-2"></i> 내일 훈련 자동 스케줄러 (처방전)</h4>
                            <div class="flex flex-col sm:flex-row gap-4">
                                <div class="flex-1 bg-gray-800 rounded p-3">
                                    <span class="text-xs font-bold text-gray-400">외인 용병 전용</span>
                                    <p class="text-sm text-white mt-1">오전 볼 훈련 제외. 하체 코어 근력 및 체력 보강 웨이트 2시간 집중 배정.</p>
                                </div>
                                <div class="flex-1 bg-gray-800 rounded p-3">
                                    <span class="text-xs font-bold text-gray-400">레프트 리시브 라인</span>
                                    <p class="text-sm text-white mt-1">서브 머신을 이용한 무회전 플로터 서브 200구 집중 리시브 훈련 (발 스텝 교정).</p>
                                </div>
                                <div class="flex-1 bg-gray-800 rounded p-3 border border-yellow-500">
                                    <span class="text-xs font-bold text-yellow-500">다음 경기 상대 (스파이더스) 대비</span>
                                    <p class="text-sm text-white mt-1">상대 주포가 라이트 공격 비율이 70%임. 레프트 블로커 2인 콤비네이션 비디오 분석 실시.</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </main>

    <script>
        lucide.createIcons();

        // 1. 탭 전환 스크립트
        function switchTab(tabId) {
            ['pre', 'in', 'post'].forEach(id => {
                document.getElementById('view-' + id).classList.add('hidden');
                document.getElementById('view-' + id).classList.remove('block');
                document.getElementById('tab-' + id).classList.remove('active', 'border-blue-500', 'text-white');
            });
            document.getElementById('view-' + tabId).classList.remove('hidden');
            document.getElementById('view-' + tabId).classList.add('block', 'animate-fade-in');
            document.getElementById('tab-' + tabId).classList.add('active', 'border-blue-500', 'text-white');
            
            // Post 탭 열릴 때 차트 렌더링
            if(tabId === 'post' && !window.radarChartCreated) {
                initRadarChart();
                window.radarChartCreated = true;
            }
        }

        // 2. 실시간 시계
        setInterval(() => {
            const now = new Date();
            document.getElementById('clock').innerText = now.toLocaleTimeString('en-US', { hour12: false });
        }, 1000);

        // 3. 경기 중(In-Match) 실시간 로그 및 모멘텀 계산
        let rhythmScore = 85;
        function logEvent(action, colorClass) {
            const logUl = document.getElementById('event-log');
            const newLi = document.createElement('li');
            newLi.className = `text-xs text-white bg-gray-800 p-2 rounded flex items-center border-l-4 border-transparent`;
            
            // 색상 연동
            if(colorClass.includes('blue')) newLi.style.borderColor = '#3b82f6';
            if(colorClass.includes('green')) newLi.style.borderColor = '#22c55e';
            if(colorClass.includes('yellow')) newLi.style.borderColor = '#eab308';
            if(colorClass.includes('red')) newLi.style.borderColor = '#ef4444';

            const timeStr = new Date().toLocaleTimeString('en-US', { hour12: false });
            newLi.innerHTML = `<span class="text-gray-500 w-16">${timeStr}</span> <span class="font-bold ml-2">${action}</span>`;
            logUl.insertBefore(newLi, logUl.firstChild);
            
            // 좋은 플레일 때 리듬 회복
            if(action.includes('성공')) { dropRhythm(-5); }
        }

        // 보좌관 획기적 아이디어: 리듬 붕괴 시 자동 타임아웃 경고
        function dropRhythm(amount) {
            rhythmScore -= amount;
            if(rhythmScore > 100) rhythmScore = 100;
            if(rhythmScore < 0) rhythmScore = 0;

            const bar = document.getElementById('rhythm-bar');
            const text = document.getElementById('rhythm-text');
            const alertBox = document.getElementById('timeout-alert');
            
            bar.style.width = rhythmScore + '%';
            
            if(rhythmScore <= 40) {
                bar.className = "bg-red-500 h-3 rounded-full transition-all duration-500";
                text.className = "text-red-400 blink-red";
                text.innerText = `위험! 붕괴 중 (${rhythmScore}%)`;
                alertBox.classList.remove('hidden'); // 타임아웃 팝업 띄우기
            } else if(rhythmScore <= 70) {
                bar.className = "bg-yellow-500 h-3 rounded-full transition-all duration-500";
                text.className = "text-yellow-400";
                text.innerText = `주의 요망 (${rhythmScore}%)`;
                alertBox.classList.add('hidden');
            } else {
                bar.className = "bg-green-500 h-3 rounded-full transition-all duration-500";
                text.className = "text-green-400";
                text.innerText = `안정적 (${rhythmScore}%)`;
                alertBox.classList.add('hidden');
            }
        }

        // 4. 경기 후(Post-Match) 레이더 차트 생성 로직 (Chart.js 활용)
        function initRadarChart() {
            const ctx = document.getElementById('radarChart').getContext('2d');
            Chart.defaults.color = '#94a3b8';
            Chart.defaults.font.family = "'Noto Sans KR', sans-serif";
            
            new Chart(ctx, {
                type: 'radar',
                data: {
                    labels: ['공격 성공률', '블로킹', '디그(수비)', '리시브 효율', '서브 에이스'],
                    datasets: [{
                        label: '오늘 우리 팀',
                        data: [55, 60, 80, 35, 50], // 리시브가 35로 찌그러진 형태
                        backgroundColor: 'rgba(59, 130, 246, 0.4)',
                        borderColor: '#3b82f6',
                        pointBackgroundColor: '#fff',
                        pointBorderColor: '#3b82f6',
                        borderWidth: 2
                    },
                    {
                        label: '리그 평균 (목표치)',
                        data: [50, 50, 50, 50, 50],
                        backgroundColor: 'rgba(156, 163, 175, 0.1)',
                        borderColor: '#64748b',
                        borderDash: [5, 5],
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    scales: {
                        r: {
                            angleLines: { color: 'rgba(255, 255, 255, 0.1)' },
                            grid: { color: 'rgba(255, 255, 255, 0.1)' },
                            pointLabels: { color: '#cbd5e1', font: { size: 11, weight: 'bold' } },
                            ticks: { display: false, min: 0, max: 100 }
                        }
                    },
                    plugins: {
                        legend: { position: 'bottom', labels: { color: '#cbd5e1' } }
                    }
                }
            });
        }
    </script>
</body>
</html>
