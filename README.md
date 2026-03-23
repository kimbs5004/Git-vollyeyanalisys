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
        /* 스크롤바 커스텀 */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-thumb { background: #475569; border-radius: 10px; }
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
            <button onclick="switchTab('pre')" id="tab-pre" class="tab-btn active px-4 py-2 text-gray-400 hover:text-white transition">1. 경기 전 (종합관리)</button>
            <button onclick="switchTab('in')" id="tab-in" class="tab-btn px-4 py-2 text-gray-400 hover:text-white transition">2. 경기 중 (AI 자동분석)</button>
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
             1. 경기 전 (평소/선수단 관리 및 종합 추천)
             ========================================== -->
        <section id="view-pre" class="block space-y-6 max-w-screen-2xl mx-auto">
            
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <!-- [입력창 1] 선수 1일 자가진단 폼 -->
                <div class="glass-panel rounded-2xl p-6 shadow-xl flex flex-col h-[500px] overflow-y-auto border-t-4 border-blue-500">
                    <h2 class="text-xl font-bold mb-4 flex items-center text-blue-400 sticky top-0 bg-slate-800/90 py-2 z-10">
                        <i data-lucide="clipboard-edit" class="w-5 h-5 mr-2"></i> [선수 본인] 일일 자가진단 입력
                    </h2>
                    <form class="space-y-4" onsubmit="event.preventDefault(); alert('선수 자가진단 완료.');">
                        <div class="grid grid-cols-2 gap-3">
                            <div><label class="text-[11px] text-gray-400">성명</label><input type="text" value="김철수" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700"></div>
                            <div><label class="text-[11px] text-gray-400">포지션</label><input type="text" value="미들블로커 (MB)" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700"></div>
                            <div><label class="text-[11px] text-gray-400">생년월일</label><input type="date" value="1999-05-12" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700"></div>
                            <div><label class="text-[11px] text-gray-400">혈액형</label><select class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700"><option>O형</option><option>A형</option><option>B형</option><option>AB형</option></select></div>
                            <div><label class="text-[11px] text-gray-400">키 (cm)</label><input type="number" value="198" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700"></div>
                            <div><label class="text-[11px] text-gray-400">몸무게 (kg)</label><input type="number" value="88" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700"></div>
                        </div>
                        <div class="grid grid-cols-2 gap-4 pt-2 border-t border-gray-700">
                            <div>
                                <label class="text-[11px] text-gray-400">정신 컨디션 (1~10)</label>
                                <input type="range" min="1" max="10" value="9" class="w-full mt-1 accent-blue-500">
                            </div>
                            <div>
                                <label class="text-[11px] text-gray-400">스트레스 지수 (1~10)</label>
                                <input type="range" min="1" max="10" value="3" class="w-full mt-1 accent-red-500">
                            </div>
                        </div>
                        <div>
                            <label class="text-[11px] text-gray-400">수면 시간 및 질</label>
                            <select class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700">
                                <option>8시간 이상 (매우 좋음 - 피로회복 완료)</option>
                                <option>6~8시간 (보통)</option>
                                <option>6시간 미만 (수면 부족)</option>
                            </select>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="text-[11px] text-gray-400">통증 부위 및 강도</label>
                                <input type="text" placeholder="예: 오른쪽 발목 미세 통증 (2/10)" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700">
                            </div>
                            <div>
                                <label class="text-[11px] text-gray-400">신체 불편 사항</label>
                                <input type="text" placeholder="예: 소화 불량 약간" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700">
                            </div>
                        </div>
                        <button class="w-full bg-blue-600 hover:bg-blue-500 text-white font-bold py-2 rounded-lg transition mt-2">자가진단 제출</button>
                    </form>
                </div>

                <!-- [입력창 2] 코칭스태프 평가창 -->
                <div class="glass-panel rounded-2xl p-6 shadow-xl flex flex-col h-[500px] overflow-y-auto border-t-4 border-green-500">
                    <h2 class="text-xl font-bold mb-4 flex items-center text-green-400 sticky top-0 bg-slate-800/90 py-2 z-10">
                        <i data-lucide="users" class="w-5 h-5 mr-2"></i> [코칭스태프] 관찰 및 선수 평가
                    </h2>
                    <form class="space-y-4" onsubmit="event.preventDefault(); alert('코칭스태프 평가 반영 완료.');">
                        <div>
                            <label class="text-[11px] text-gray-400">평가 대상 선수 선택</label>
                            <select class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700">
                                <option>김철수 (MB) - 데이터 매칭 중</option>
                            </select>
                        </div>
                        <div class="grid grid-cols-2 gap-4 pt-2 border-t border-gray-700">
                            <div>
                                <label class="text-[11px] text-gray-400">코치진 관찰 컨디션</label>
                                <select class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700">
                                    <option>최상 (몸놀림 아주 가벼움)</option><option>보통</option><option>저하 (피로 누적 보임)</option>
                                </select>
                            </div>
                            <div>
                                <label class="text-[11px] text-gray-400">스타팅 추천 여부</label>
                                <select class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-green-500">
                                    <option>강력 추천 (스타팅 포함)</option><option>백업 대기 (조커 활용)</option><option>휴식 권장</option>
                                </select>
                            </div>
                        </div>
                        <div>
                            <label class="text-[11px] text-gray-400">관찰된 불편사항 / 메디컬 체크 결괏값</label>
                            <textarea class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700" rows="2" placeholder="선수 보고와 일치함. 발목 테이핑만으로 출전 지장 없음."></textarea>
                        </div>
                        <div>
                            <label class="text-[11px] text-gray-400">종합 코멘트 (연습/면담 기반)</label>
                            <textarea class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700" rows="3" placeholder="어제 면담 시 멘탈 매우 긍정적. 블로킹 손모양 교정 완료되어 감각 최고조."></textarea>
                        </div>
                        <button class="w-full bg-green-600 hover:bg-green-500 text-white font-bold py-2 rounded-lg transition mt-2">평가서 제출 및 AI 융합</button>
                    </form>
                </div>
            </div>

            <!-- [통합 결과] 선수+스태프 융합 AI 스타팅 추천 & 검증 -->
            <div class="glass-panel rounded-2xl p-6 shadow-xl border-t-4 border-yellow-400">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-2xl font-black flex items-center text-yellow-400">
                        <i data-lucide="cpu" class="w-6 h-6 mr-2"></i> AI 통합 스타팅 6인 및 분석 검증
                    </h2>
                    <span class="bg-gray-800 px-3 py-1 rounded text-sm text-gray-300 border border-gray-700">다음 매치업: 챔피언스</span>
                </div>
                
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <!-- AI 검증 및 분석 패널 -->
                    <div class="lg:col-span-2 space-y-4">
                        <div class="bg-gray-800/80 p-5 rounded-xl border border-gray-700">
                            <h3 class="text-blue-400 font-bold mb-3 flex items-center"><i data-lucide="check-circle" class="w-4 h-4 mr-2"></i> 선수 데이터 + 코칭스태프 종합 분석 결과</h3>
                            <ul class="space-y-3 text-sm text-gray-300">
                                <li class="flex items-start bg-gray-900 p-3 rounded">
                                    <span class="bg-blue-600 px-2 py-0.5 rounded text-[10px] font-bold mr-3 mt-0.5 whitespace-nowrap">선발 확정</span>
                                    <div>
                                        <strong class="text-white">김철수 (MB) - 적합도 98%</strong><br>
                                        <span class="text-gray-400">선수 자가진단(스트레스 3/수면 8h 최상)과 코치진 평가(블로킹 감각 최상) 100% 일치. 상대 주포의 레프트 공격(비중 60%)을 차단할 최적의 카드.</span>
                                    </div>
                                </li>
                                <li class="flex items-start bg-gray-900 p-3 rounded">
                                    <span class="bg-red-600 px-2 py-0.5 rounded text-[10px] font-bold mr-3 mt-0.5 whitespace-nowrap">모니터링</span>
                                    <div>
                                        <strong class="text-white">용병 A (OP) - 적합도 75%</strong><br>
                                        <span class="text-gray-400">선수 본인이 우측 무릎 미세 통증(3/10) 보고. 코칭스태프 평가 '백업 대기' 요망. 선발 출전시키되 점프 타점 실시간 모니터링 후 국내 라이트 즉각 교체 준비.</span>
                                    </div>
                                </li>
                            </ul>
                        </div>
                    </div>

                    <!-- 코트 그래픽 라인업 -->
                    <div class="lg:col-span-1 bg-green-900/20 border-2 border-green-500/30 rounded-xl relative overflow-hidden flex items-center justify-center p-4 h-[300px]">
                        <div class="absolute inset-4 border-2 border-white/20"></div>
                        <div class="absolute inset-x-4 top-1/3 border-b-2 border-white/20"></div>
                        
                        <div class="grid grid-cols-3 gap-x-8 gap-y-12 w-full max-w-sm z-10 relative">
                            <!-- 전위 -->
                            <div class="flex flex-col items-center">
                                <div class="w-10 h-10 rounded-full bg-blue-600 flex items-center justify-center font-bold border-2 border-green-400">OH</div>
                                <span class="text-[11px] mt-1 text-center whitespace-nowrap">김배구<br>(🟢95%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-10 h-10 rounded-full bg-blue-600 flex items-center justify-center font-bold border-2 border-green-400 shadow-[0_0_15px_rgba(74,222,128,0.8)]">MB</div>
                                <span class="text-[11px] mt-1 text-yellow-300 font-bold text-center whitespace-nowrap">김철수<br>(🟢98%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-10 h-10 rounded-full bg-blue-600 flex items-center justify-center font-bold border-2 border-yellow-400">OP</div>
                                <span class="text-[11px] mt-1 text-red-300 text-center whitespace-nowrap">용병A<br>(🟡75%)</span>
                            </div>
                            <!-- 후위 -->
                            <div class="flex flex-col items-center">
                                <div class="w-10 h-10 rounded-full bg-blue-800 flex items-center justify-center font-bold border-2 border-green-400">OH</div>
                                <span class="text-[11px] mt-1 text-center whitespace-nowrap">이수비<br>(🟢90%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-10 h-10 rounded-full bg-gray-600 flex items-center justify-center font-bold border-2 border-green-400">L</div>
                                <span class="text-[11px] mt-1 text-center whitespace-nowrap">박디그<br>(🟢88%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-10 h-10 rounded-full bg-blue-800 flex items-center justify-center font-bold border-2 border-green-400">S</div>
                                <span class="text-[11px] mt-1 text-center whitespace-nowrap">최세터<br>(🟢92%)</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- ==========================================
             2. 경기 중 (완전 자동화 비전 분석 및 실시간 작전)
             ========================================== -->
        <section id="view-in" class="hidden space-y-6 h-full max-w-screen-2xl mx-auto">
            
            <div class="bg-blue-900/50 border border-blue-500 text-blue-200 p-4 rounded-xl flex items-center shadow-lg">
                <i data-lucide="info" class="w-6 h-6 mr-3 text-blue-400 shrink-0"></i>
                <p class="text-sm">수동 입력을 전면 폐지했습니다. <strong>AI 비전(Computer Vision) 카메라가 중계방송 및 경기장 상황을 스스로 추적하여 실시간 공격/블로킹/범실 데이터를 자동 기록</strong>하고 벤치에 즉각 알림을 보냅니다. (아래 로그가 자동으로 올라가는 것을 확인하십시오.)</p>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-4 gap-6 h-[70vh]">
                
                <!-- AI 비전 자동 분석 영상 피드 (좌측 3칸) -->
                <div class="lg:col-span-3 flex flex-col space-y-4">
                    <!-- AI 카메라 영상 송출창 -->
                    <div class="bg-black rounded-2xl border-2 border-gray-700 h-full relative overflow-hidden flex items-center justify-center shadow-2xl">
                        <!-- 영상 대체 이미지 -->
                        <img src="https://images.unsplash.com/photo-1612872087720-bb876e2e67d1?ixlib=rb-4.0.3&auto=format&fit=crop&w=1200&q=80" class="absolute inset-0 w-full h-full object-cover opacity-30">
                        
                        <!-- AI Vision Tagging 그래픽 요소 (움직이는 타겟) -->
                        <div class="absolute inset-0 bg-[url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0MCIgaGVpZ2h0PSI0MCI+PHBhdGggZD0iTTAgMGg0MHY0MEgweiIgZmlsbD0ibm9uZSIvPjxwYXRoIGQ0iTTAgMGgxdjQwSDB6TTAgMGg0MHYxSDB6IiBmaWxsPSJyZ2JhKDI1NSwyNTUsMjU1LDAuMSkiLz48L3N2Zz4=')] opacity-50"></div>
                        
                        <!-- 추적 박스 UI -->
                        <div id="ai-tracker-box" class="absolute top-1/3 left-1/2 transform -translate-x-1/2 -translate-y-1/2 border-2 border-green-500 w-48 h-56 rounded transition-all duration-1000 flex flex-col justify-end p-2 shadow-[0_0_20px_rgba(34,197,94,0.4)]">
                            <div class="bg-black/70 p-1 rounded">
                                <span id="tracker-player" class="bg-green-500 text-white text-[11px] px-2 py-0.5 rounded font-bold w-max block mb-1">우리팀 공격 준비 (추적중)</span>
                                <span id="tracker-data" class="text-green-300 text-[10px] font-mono">분석 대기중...</span>
                            </div>
                        </div>

                        <!-- 상단 상태 바 -->
                        <div class="absolute top-4 left-4 bg-red-600 text-white px-4 py-2 rounded-full text-sm font-bold flex items-center shadow-lg border border-red-400">
                            <span class="w-3 h-3 bg-white rounded-full animate-ping mr-3"></span> AI VISION AUTO-TRACKING ON
                        </div>

                        <!-- 하단 스캐닝 이펙트 -->
                        <div class="absolute bottom-0 w-full h-1 bg-green-500/50 shadow-[0_0_15px_#22c55e]"></div>
                    </div>
                </div>

                <!-- 실시간 AI 자동 로그 및 벤치 상황판 (우측 1칸) -->
                <div class="glass-panel rounded-2xl p-5 lg:col-span-1 flex flex-col h-full overflow-hidden shadow-2xl border border-gray-700">
                    <h2 class="text-lg font-black mb-4 border-b border-gray-700 pb-2 text-white flex items-center">
                        <i data-lucide="zap" class="w-5 h-5 mr-2 text-yellow-400"></i> AI 벤치 자동 알림
                    </h2>
                    
                    <!-- 팀 모멘텀(리듬) 게이지 -->
                    <div class="mb-6 bg-gray-800/80 p-3 rounded-lg border border-gray-700">
                        <div class="flex justify-between text-xs font-bold mb-2">
                            <span class="text-gray-300">현재 팀 리듬 (모멘텀)</span>
                            <span id="rhythm-text" class="text-green-400 text-sm">안정적 (100%)</span>
                        </div>
                        <div class="w-full bg-gray-900 rounded-full h-4 border border-gray-600 overflow-hidden shadow-inner">
                            <div id="rhythm-bar" class="bg-green-500 h-4 rounded-full transition-all duration-700" style="width: 100%"></div>
                        </div>
                    </div>

                    <!-- 타임아웃 경고 모달창 (AI 자동 호출용) -->
                    <div id="timeout-alert" class="hidden bg-red-600 text-white p-4 rounded-xl mb-4 shadow-[0_0_20px_rgba(239,68,68,0.8)] blink-red border-2 border-white">
                        <div class="flex items-center justify-center mb-1"><i data-lucide="siren" class="w-6 h-6 mr-2"></i><span class="font-black text-lg">작전 타임 즉각 권장!</span></div>
                        <p class="text-[11px] text-center font-bold">어이없는 범실 누적으로 팀 리듬 완전 붕괴.</p>
                    </div>

                    <!-- 실시간 자동 이벤트 로그 (사람 입력 없음) -->
                    <div class="flex-1 overflow-hidden flex flex-col">
                        <div class="flex justify-between items-end mb-2">
                            <h3 class="text-xs font-bold text-blue-300 flex items-center"><i data-lucide="activity" class="w-4 h-4 mr-1"></i> AI 비전 자동 기록 스트림</h3>
                            <span class="text-[10px] text-gray-500 animate-pulse">실시간 수신중...</span>
                        </div>
                        <div class="flex-1 bg-black/50 border border-gray-700 rounded-lg p-2 overflow-y-auto" id="auto-log-container">
                            <ul id="event-log" class="space-y-2 flex flex-col justify-end min-h-full">
                                <!-- JS로 자동 생성됨 -->
                            </ul>
                        </div>
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
                                <div class="flex-1 bg-gray-800 rounded p-3 shadow-inner">
                                    <span class="text-xs font-bold text-gray-400">외인 용병 전용</span>
                                    <p class="text-sm text-white mt-1">오전 볼 훈련 제외. 하체 코어 근력 및 체력 보강 웨이트 2시간 집중 배정.</p>
                                </div>
                                <div class="flex-1 bg-gray-800 rounded p-3 shadow-inner">
                                    <span class="text-xs font-bold text-gray-400">레프트 리시브 라인</span>
                                    <p class="text-sm text-white mt-1">서브 머신을 이용한 무회전 플로터 서브 200구 집중 리시브 훈련 (발 스텝 교정).</p>
                                </div>
                                <div class="flex-1 bg-gray-800 rounded p-3 border border-yellow-500 shadow-inner">
                                    <span class="text-xs font-bold text-yellow-500">다음 경기 상대 대비</span>
                                    <p class="text-sm text-white mt-1">상대 주포가 라이트 공격 비율이 70%임. 레프트 블로커 2인 콤비네이션 비디오 분석.</p>
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
            
            // Post 탭 열릴 때 차트 생성
            if(tabId === 'post' && !window.radarChartCreated) {
                initRadarChart();
                window.radarChartCreated = true;
            }
            
            // In 탭 열릴 때 AI 자동 시뮬레이션 시작
            if(tabId === 'in' && !window.autoSimStarted) {
                startAIAutoSimulation();
                window.autoSimStarted = true;
            }
        }

        // 시계
        setInterval(() => {
            const now = new Date();
            document.getElementById('clock').innerText = now.toLocaleTimeString('en-US', { hour12: false });
        }, 1000);

        /* ==============================================================
           경기 중 완전 자동화(AI Vision Auto-Tracking) 시뮬레이션 로직
           ============================================================== */
        let rhythmScore = 100;
        const autoEvents = [
            { action: "김철수 A속공 성공", detail: "속도 95km/h, 블로킹 완전 따돌림", color: "blue", rhythm: +5, border: "border-blue-500", tracker: {x: "30%", y:"40%", color: "green", text: "속공 득점 인식"} },
            { action: "상대 평범한 서브", detail: "우리팀 완벽한 리시브 찬스 발생", color: "yellow", rhythm: -15, border: "border-yellow-500", tracker: {x: "70%", y:"20%", color: "yellow", text: "약한 서브 궤적"} },
            { action: "이수비 리시브 실패 (범실)", detail: "어이없는 사인 미스로 리듬 저하", color: "red", rhythm: -25, border: "border-red-500", tracker: {x: "20%", y:"80%", color: "red", text: "사인 미스 감지!"} },
            { action: "용병A 스파이크 아웃 (범실)", detail: "타점 12cm 하락. 체력 저하 의심.", color: "red", rhythm: -20, border: "border-red-500", tracker: {x: "80%", y:"30%", color: "red", text: "타점 저하 경고"} },
            { action: "박디그 환상적인 디그", detail: "팀 분위기 상승", color: "green", rhythm: +10, border: "border-green-500", tracker: {x: "40%", y:"85%", color: "blue", text: "디그 모션 인식"} }
        ];

        function startAIAutoSimulation() {
            let eventCount = 0;
            
            // 3.5초마다 AI가 영상을 자동 분석해서 로그를 쏴주는 시뮬레이션
            setInterval(() => {
                if(document.getElementById('view-in').classList.contains('hidden')) return; // 탭 가려지면 정지
                
                const evt = autoEvents[eventCount % autoEvents.length];
                
                // 1. 영상 위 트래커(박스) 위치 및 색상 자동 이동 효과
                const tracker = document.getElementById('ai-tracker-box');
                const tPlayer = document.getElementById('tracker-player');
                const tData = document.getElementById('tracker-data');
                
                tracker.style.left = evt.tracker.x;
                tracker.style.top = evt.tracker.y;
                tracker.className = `absolute transform -translate-x-1/2 -translate-y-1/2 border-2 w-40 h-48 rounded transition-all duration-1000 flex flex-col justify-end p-2 border-${evt.color}-500 shadow-[0_0_20px_rgba(0,0,0,0.8)] z-10 bg-${evt.color}-900/20`;
                
                tPlayer.className = `bg-${evt.color}-600 text-white text-[11px] px-2 py-0.5 rounded font-bold w-max block mb-1`;
                tPlayer.innerText = evt.tracker.text;
                tData.innerText = evt.detail;
                tData.className = `text-${evt.color}-300 text-[10px] font-bold`;

                // 2. 우측 이벤트 로그 자동 추가
                addAutoLog(evt.action, evt.detail, evt.border);
                
                // 3. 리듬 점수 자동 계산 및 경고창
                updateAutoRhythm(evt.rhythm);

                eventCount++;
            }, 3500);
        }

        function addAutoLog(action, detail, borderClass) {
            const logUl = document.getElementById('event-log');
            const newLi = document.createElement('li');
            newLi.className = `text-xs text-white bg-gray-800 p-2.5 rounded-lg flex flex-col border-l-4 ${borderClass} animate-fade-in shadow-md`;
            
            const timeStr = new Date().toLocaleTimeString('en-US', { hour12: false });
            newLi.innerHTML = `
                <div class="flex items-center">
                    <span class="text-gray-400 font-mono w-16 text-[10px]">${timeStr}</span> 
                    <span class="font-bold text-[13px] ml-1">${action}</span>
                </div>
                <span class="text-[10px] text-gray-400 mt-1 ml-17">${detail}</span>
            `;
            
            logUl.appendChild(newLi);
            
            // 로그 창 자동 스크롤 (항상 최신 내용이 보이게)
            const container = document.getElementById('auto-log-container');
            container.scrollTop = container.scrollHeight;
            
            // 메모리 관리를 위해 로그 10개 넘으면 오래된 것 삭제
            if(logUl.children.length > 10) logUl.removeChild(logUl.firstChild);
        }

        function updateAutoRhythm(amount) {
            rhythmScore += amount;
            if(rhythmScore > 100) rhythmScore = 100;
            if(rhythmScore < 0) rhythmScore = 0;

            const bar = document.getElementById('rhythm-bar');
            const text = document.getElementById('rhythm-text');
            const alertBox = document.getElementById('timeout-alert');
            
            bar.style.width = rhythmScore + '%';
            
            if(rhythmScore <= 40) {
                bar.className = "bg-red-500 h-4 rounded-full transition-all duration-300";
                text.className = "text-red-400 blink-red font-black";
                text.innerText = `위험! 붕괴 중 (${rhythmScore}%)`;
                alertBox.classList.remove('hidden'); // 타임아웃 자동 팝업
            } else if(rhythmScore <= 70) {
                bar.className = "bg-yellow-500 h-4 rounded-full transition-all duration-300";
                text.className = "text-yellow-400 font-bold";
                text.innerText = `주의 요망 (${rhythmScore}%)`;
                alertBox.classList.add('hidden');
            } else {
                bar.className = "bg-green-500 h-4 rounded-full transition-all duration-300";
                text.className = "text-green-400 font-bold";
                text.innerText = `안정적 (${rhythmScore}%)`;
                alertBox.classList.add('hidden');
            }
        }

        // 경기 후 차트
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
                        data: [55, 60, 80, 35, 50],
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
