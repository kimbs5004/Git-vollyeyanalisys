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
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-thumb { background: #475569; border-radius: 10px; }
    </style>
</head>
<body class="h-screen flex flex-col overflow-hidden relative">

    <!-- ==========================================
         [보안] 접속 권한 분리 로그인 화면 (RBAC)
         ========================================== -->
    <div id="login-screen" class="fixed inset-0 bg-gray-900 z-[200] flex flex-col items-center justify-center px-4">
        <div class="mb-8 text-center">
            <div class="bg-blue-600 p-4 rounded-2xl inline-block mb-4 shadow-[0_0_30px_rgba(37,99,235,0.5)]">
                <i data-lucide="activity" class="w-12 h-12 text-white"></i>
            </div>
            <h1 class="text-4xl font-black tracking-wider text-white">V-MASTER</h1>
            <p class="text-blue-400 mt-2 font-bold tracking-widest text-sm">통합 전력분석 AI 시스템 (보안 접속)</p>
        </div>

        <div class="w-full max-w-sm space-y-4">
            <button onclick="login('PLAYER')" class="w-full bg-gray-800 hover:bg-gray-700 border-2 border-gray-600 text-white font-bold py-4 rounded-xl transition flex items-center justify-center shadow-lg">
                <i data-lucide="user" class="w-5 h-5 mr-3 text-gray-400"></i> 선수용 접속 (자가진단)
            </button>
            <button onclick="login('COACH')" class="w-full bg-gray-800 hover:bg-green-800 border-2 border-green-600 text-white font-bold py-4 rounded-xl transition flex items-center justify-center shadow-lg">
                <i data-lucide="users" class="w-5 h-5 mr-3 text-green-400"></i> 코칭스태프용 접속 (평가입력)
            </button>
            <button onclick="login('ANALYST')" class="w-full bg-blue-700 hover:bg-blue-600 border-2 border-blue-400 text-white font-bold py-4 rounded-xl transition flex items-center justify-center shadow-[0_0_20px_rgba(59,130,246,0.6)]">
                <i data-lucide="shield-alert" class="w-5 h-5 mr-3 text-white"></i> 전력분석관 전용 (마스터 권한)
            </button>
        </div>
        <p class="mt-8 text-xs text-gray-500">※ 본 시스템은 특허 출원 예정이며, 무단 유출을 엄격히 금지합니다.</p>
    </div>

    <!-- Top Navigation (로그인 후 권한별로 노출 다름) -->
    <header class="bg-gray-900 border-b border-gray-800 p-4 flex justify-between items-center shadow-lg shrink-0">
        <div class="flex items-center space-x-3">
            <div class="bg-blue-600 p-2 rounded-lg hidden sm:block"><i data-lucide="activity" class="w-5 h-5 text-white"></i></div>
            <h1 class="text-xl font-black tracking-wider text-white">V-MASTER</h1>
            <span id="role-badge" class="bg-gray-700 px-2 py-0.5 rounded text-[10px] font-bold text-gray-300 ml-2 border border-gray-600">권한대기</span>
        </div>
        
        <!-- 권한별 탭 메뉴 -->
        <div id="nav-tabs" class="hidden md:flex space-x-1 overflow-x-auto">
            <button onclick="switchTab('pre-player')" id="tab-pre-player" class="tab-btn hidden px-3 py-2 text-sm text-gray-400 hover:text-white whitespace-nowrap">일일 자가진단</button>
            <button onclick="switchTab('pre-coach')" id="tab-pre-coach" class="tab-btn hidden px-3 py-2 text-sm text-gray-400 hover:text-white whitespace-nowrap">선수단 평가</button>
            
            <button onclick="switchTab('pre-master')" id="tab-pre-master" class="tab-btn hidden px-3 py-2 text-sm text-gray-400 hover:text-white whitespace-nowrap">1. 종합관리(Pre)</button>
            <button onclick="switchTab('in')" id="tab-in" class="tab-btn hidden px-3 py-2 text-sm text-gray-400 hover:text-white whitespace-nowrap">2. 실시간 AI(In)</button>
            <button onclick="switchTab('post')" id="tab-post" class="tab-btn hidden px-3 py-2 text-sm text-gray-400 hover:text-white whitespace-nowrap">3. 분석서(Post)</button>
            <button onclick="switchTab('scout')" id="tab-scout" class="tab-btn hidden px-3 py-2 text-sm text-yellow-500 hover:text-yellow-400 whitespace-nowrap">4. 용병선발</button>
        </div>

        <div class="flex items-center space-x-4">
            <button onclick="logout()" class="text-xs text-gray-400 hover:text-white underline hidden sm:block">로그아웃</button>
            <span class="text-xs text-gray-400 flex items-center hidden sm:flex"><i data-lucide="clock" class="w-4 h-4 mr-1"></i> <span id="clock">19:00</span></span>
        </div>
    </header>

    <!-- 모바일 하단 네비게이션 (모바일 전용) -->
    <div id="mobile-nav" class="md:hidden bg-gray-900 border-t border-gray-800 flex justify-around p-2 fixed bottom-0 w-full z-50 hidden">
        <!-- JS로 권한에 맞게 버튼이 채워집니다. -->
    </div>

    <!-- Main Content Area -->
    <main class="flex-1 overflow-y-auto p-4 sm:p-6 pb-20 md:pb-6">
        
        <!-- ==========================================
             [선수 전용] 1. 일일 자가진단 폼
             ========================================== -->
        <section id="view-pre-player" class="hidden space-y-6 max-w-lg mx-auto">
            <div class="glass-panel rounded-2xl p-6 shadow-xl flex flex-col border-t-4 border-blue-500">
                <h2 class="text-lg font-bold mb-4 flex items-center text-blue-400">
                    <i data-lucide="clipboard-edit" class="w-5 h-5 mr-2"></i> 일일 컨디션 자가진단
                </h2>
                <form class="space-y-4" onsubmit="event.preventDefault(); showToast('데이터가 코칭스태프에게 안전하게 전송되었습니다.');">
                    <div class="grid grid-cols-2 gap-3">
                        <div><label class="text-[11px] text-gray-400">성명</label><input type="text" value="김철수" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700" readonly></div>
                        <div><label class="text-[11px] text-gray-400">포지션</label><input type="text" value="MB" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700" readonly></div>
                        <div><label class="text-[11px] text-gray-400">체중 (kg) 변화</label><input type="number" value="88" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700"></div>
                        <div><label class="text-[11px] text-gray-400">수면 시간</label><select class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700"><option>8시간 이상</option><option>6~8시간</option><option>6시간 미만</option></select></div>
                    </div>
                    <div class="pt-2 border-t border-gray-700">
                        <label class="text-[11px] text-gray-400 flex justify-between"><span>정신 컨디션 (최악1 ~ 최상10)</span><span id="mental-val" class="text-blue-400 font-bold">9</span></label>
                        <input type="range" min="1" max="10" value="9" class="w-full mt-1 accent-blue-500" oninput="document.getElementById('mental-val').innerText = this.value">
                    </div>
                    <div>
                        <label class="text-[11px] text-gray-400 flex justify-between"><span>스트레스 지수 (없음1 ~ 극심10)</span><span id="stress-val" class="text-red-400 font-bold">3</span></label>
                        <input type="range" min="1" max="10" value="3" class="w-full mt-1 accent-red-500" oninput="document.getElementById('stress-val').innerText = this.value">
                    </div>
                    <div class="pt-2 border-t border-gray-700">
                        <label class="text-[11px] text-gray-400">통증 부위 및 강도 (비밀보장)</label>
                        <input type="text" placeholder="예: 오른쪽 발목 미세 통증" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700">
                    </div>
                    <div>
                        <label class="text-[11px] text-gray-400">기타 신체 불편 사항</label>
                        <textarea rows="2" placeholder="의료진/코칭스태프에게 전달할 내용" class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700"></textarea>
                    </div>
                    <button class="w-full bg-blue-600 hover:bg-blue-500 text-white font-bold py-3 rounded-lg transition mt-4 shadow-[0_0_15px_rgba(37,99,235,0.4)]">자가진단 데이터 전송</button>
                    <button type="button" onclick="logout()" class="w-full bg-gray-800 text-gray-400 font-bold py-2 rounded-lg mt-2 md:hidden">로그아웃</button>
                </form>
            </div>
        </section>

        <!-- ==========================================
             [코칭스태프 전용] 1. 선수단 관찰 평가
             ========================================== -->
        <section id="view-pre-coach" class="hidden space-y-6 max-w-lg mx-auto">
            <div class="glass-panel rounded-2xl p-6 shadow-xl flex flex-col border-t-4 border-green-500">
                <h2 class="text-lg font-bold mb-4 flex items-center text-green-400">
                    <i data-lucide="users" class="w-5 h-5 mr-2"></i> 선수단 관찰 및 현장 평가
                </h2>
                <div class="bg-gray-800/50 p-3 rounded-lg border border-gray-700 mb-4 text-xs text-gray-300">
                    <i data-lucide="info" class="w-4 h-4 inline mr-1 text-blue-400"></i> 코칭스태프 입력 자료는 분석관의 AI 데이터와 융합되어 최종 스타팅 라인업 추천에 반영됩니다.
                </div>
                <form class="space-y-4" onsubmit="event.preventDefault(); showToast('평가 데이터가 전력분석 마스터에게 전송되었습니다.');">
                    <div>
                        <label class="text-[11px] text-gray-400">평가 대상 선수 선택</label>
                        <select class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-green-500">
                            <option>김철수 (MB) - 선수입력 완료</option>
                            <option>용병A (OP) - 선수입력 완료</option>
                        </select>
                    </div>
                    <div class="grid grid-cols-2 gap-4 pt-2 border-t border-gray-700">
                        <div>
                            <label class="text-[11px] text-gray-400">현장 관찰 컨디션</label>
                            <select class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700">
                                <option>최상 (몸놀림 가벼움)</option><option>보통</option><option>저하 (피로 보임)</option>
                            </select>
                        </div>
                        <div>
                            <label class="text-[11px] text-gray-400">코치진 스타팅 추천</label>
                            <select class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700">
                                <option>강력 추천 (스타팅)</option><option>백업 대기 (조커)</option><option>휴식 권장</option>
                            </select>
                        </div>
                    </div>
                    <div>
                        <label class="text-[11px] text-gray-400">현장 관찰 불편사항 / 메디컬 체크</label>
                        <textarea class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700" rows="2" placeholder="발목 테이핑 실시. 점프에 지장 없음."></textarea>
                    </div>
                    <div>
                        <label class="text-[11px] text-gray-400">종합 코멘트 (훈련/면담 기반)</label>
                        <textarea class="w-full bg-gray-800 rounded p-2 text-sm text-white border border-gray-700" rows="3" placeholder="블로킹 폼 교정 완료. 멘탈 매우 좋음."></textarea>
                    </div>
                    <button class="w-full bg-green-600 hover:bg-green-500 text-white font-bold py-3 rounded-lg transition mt-4 shadow-[0_0_15px_rgba(34,197,94,0.4)]">현장 평가 전송</button>
                    <button type="button" onclick="logout()" class="w-full bg-gray-800 text-gray-400 font-bold py-2 rounded-lg mt-2 md:hidden">로그아웃</button>
                </form>
            </div>
        </section>

        <!-- ==========================================
             [분석관(마스터)] 1. 종합관리 (선수+스태프 AI 융합)
             ========================================== -->
        <section id="view-pre-master" class="hidden space-y-6 max-w-screen-xl mx-auto">
            <div class="glass-panel rounded-2xl p-4 sm:p-6 shadow-xl border-t-4 border-yellow-400">
                <div class="flex justify-between items-center mb-6 border-b border-gray-700 pb-4">
                    <h2 class="text-xl sm:text-2xl font-black flex items-center text-yellow-400">
                        <i data-lucide="cpu" class="w-6 h-6 mr-2"></i> AI 통합 스타팅 6인 검증 리포트
                    </h2>
                    <span class="bg-gray-800 px-3 py-1 rounded text-xs text-gray-300 border border-gray-700">다음 상대: 챔피언스</span>
                </div>
                
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <!-- AI 검증 데이터 -->
                    <div class="lg:col-span-2 space-y-4">
                        <div class="bg-gray-800/80 p-4 sm:p-5 rounded-xl border border-gray-700">
                            <h3 class="text-blue-400 font-bold mb-3 text-sm flex items-center"><i data-lucide="database" class="w-4 h-4 mr-2"></i> 선수 자가진단 + 코칭스태프 현장평가 융합 데이터</h3>
                            <ul class="space-y-3 text-xs sm:text-sm text-gray-300">
                                <li class="flex items-start bg-gray-900 p-3 rounded">
                                    <span class="bg-blue-600 px-2 py-0.5 rounded text-[10px] font-bold mr-3 mt-0.5 whitespace-nowrap">선발 확정</span>
                                    <div>
                                        <strong class="text-white text-sm">김철수 (MB) - AI 매칭 적합도 98%</strong><br>
                                        <span class="text-gray-400 leading-relaxed">선수 본인 통증 없음 보고 및 코치진 '블로킹 폼 최상' 평가 일치. 상대 주포의 레프트 공격(비중 60%)을 방어할 최적의 픽.</span>
                                    </div>
                                </li>
                                <li class="flex items-start bg-gray-900 p-3 rounded">
                                    <span class="bg-red-600 px-2 py-0.5 rounded text-[10px] font-bold mr-3 mt-0.5 whitespace-nowrap">모니터링</span>
                                    <div>
                                        <strong class="text-white text-sm">용병 A (OP) - AI 매칭 적합도 75%</strong><br>
                                        <span class="text-gray-400 leading-relaxed">선수 본인 우측 무릎 미세 통증(3/10) 보고. 코칭스태프 '백업 권장'. 선발 출전 시 경기 중 실시간 점프 타점 분석 요망. 국내 라이트 즉각 대기.</span>
                                    </div>
                                </li>
                            </ul>
                        </div>
                    </div>

                    <!-- 코트 그래픽 라인업 -->
                    <div class="lg:col-span-1 bg-green-900/20 border-2 border-green-500/30 rounded-xl relative overflow-hidden flex items-center justify-center p-4 h-[250px] sm:h-[300px]">
                        <div class="absolute inset-4 border-2 border-white/20"></div>
                        <div class="absolute inset-x-4 top-1/3 border-b-2 border-white/20"></div>
                        
                        <div class="grid grid-cols-3 gap-x-6 sm:gap-x-8 gap-y-10 sm:gap-y-12 w-full max-w-sm z-10 relative">
                            <!-- 전위 -->
                            <div class="flex flex-col items-center">
                                <div class="w-8 h-8 sm:w-10 sm:h-10 rounded-full bg-blue-600 flex items-center justify-center font-bold text-xs border-2 border-green-400">OH</div>
                                <span class="text-[10px] sm:text-[11px] mt-1 text-center whitespace-nowrap">김배구<br>(🟢95%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-8 h-8 sm:w-10 sm:h-10 rounded-full bg-blue-600 flex items-center justify-center font-bold text-xs border-2 border-green-400 shadow-[0_0_15px_rgba(74,222,128,0.8)]">MB</div>
                                <span class="text-[10px] sm:text-[11px] mt-1 text-yellow-300 font-bold text-center whitespace-nowrap">김철수<br>(🟢98%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-8 h-8 sm:w-10 sm:h-10 rounded-full bg-blue-600 flex items-center justify-center font-bold text-xs border-2 border-yellow-400">OP</div>
                                <span class="text-[10px] sm:text-[11px] mt-1 text-red-300 text-center whitespace-nowrap">용병A<br>(🟡75%)</span>
                            </div>
                            <!-- 후위 -->
                            <div class="flex flex-col items-center">
                                <div class="w-8 h-8 sm:w-10 sm:h-10 rounded-full bg-blue-800 flex items-center justify-center font-bold text-xs border-2 border-green-400">OH</div>
                                <span class="text-[10px] sm:text-[11px] mt-1 text-center whitespace-nowrap">이수비<br>(🟢90%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-8 h-8 sm:w-10 sm:h-10 rounded-full bg-gray-600 flex items-center justify-center font-bold text-xs border-2 border-green-400">L</div>
                                <span class="text-[10px] sm:text-[11px] mt-1 text-center whitespace-nowrap">박디그<br>(🟢88%)</span>
                            </div>
                            <div class="flex flex-col items-center">
                                <div class="w-8 h-8 sm:w-10 sm:h-10 rounded-full bg-blue-800 flex items-center justify-center font-bold text-xs border-2 border-green-400">S</div>
                                <span class="text-[10px] sm:text-[11px] mt-1 text-center whitespace-nowrap">최세터<br>(🟢92%)</span>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- [1번 지시사항 조치] 선수단 전체 로스터 현황판 추가 -->
                <div class="mt-6 bg-gray-800/50 rounded-xl p-4 border border-gray-700">
                    <h3 class="text-sm font-bold mb-3 text-blue-300 flex items-center"><i data-lucide="list" class="w-4 h-4 mr-2"></i> 선수단 전체 인적사항 및 당일 컨디션 현황</h3>
                    <div class="overflow-x-auto">
                        <table class="w-full text-xs text-left text-gray-300 whitespace-nowrap">
                            <thead class="text-gray-400 bg-gray-900 uppercase border-b border-gray-700">
                                <tr>
                                    <th class="px-3 py-2">포지션</th>
                                    <th class="px-3 py-2">성명</th>
                                    <th class="px-3 py-2">신체조건 (키/몸무게/혈액형)</th>
                                    <th class="px-3 py-2">자가진단 (컨디션/수면)</th>
                                    <th class="px-3 py-2">코치진 현장 평가</th>
                                    <th class="px-3 py-2 text-center">AI 추천</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr class="border-b border-gray-800 hover:bg-gray-700/50">
                                    <td class="px-3 py-2 font-bold text-blue-400">MB</td>
                                    <td class="px-3 py-2 text-white font-bold">김철수</td>
                                    <td class="px-3 py-2">198cm / 88kg / O형</td>
                                    <td class="px-3 py-2"><span class="text-green-400">최상(9)</span> / 8h 이상</td>
                                    <td class="px-3 py-2">블로킹 폼 최상 (스타팅 권장)</td>
                                    <td class="px-3 py-2 text-center"><span class="bg-blue-600 text-white px-2 py-0.5 rounded">선발 확정</span></td>
                                </tr>
                                <tr class="border-b border-gray-800 hover:bg-gray-700/50">
                                    <td class="px-3 py-2 font-bold text-red-400">OP</td>
                                    <td class="px-3 py-2 text-white font-bold">용병A</td>
                                    <td class="px-3 py-2">205cm / 95kg / A형</td>
                                    <td class="px-3 py-2"><span class="text-yellow-400">보통(6)</span> / 6~8h</td>
                                    <td class="px-3 py-2">우측 무릎 미세 통증 호소</td>
                                    <td class="px-3 py-2 text-center"><span class="bg-yellow-600 text-white px-2 py-0.5 rounded">백업 대기</span></td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>

            </div>
        </section>

        <!-- ==========================================
             [분석관(마스터)] 2. 경기 중 (완전 자동화 분석)
             ========================================== -->
        <section id="view-in" class="hidden space-y-6 h-full max-w-screen-2xl mx-auto">
            
            <!-- [2번 지시사항 조치] 실시간 카메라/중계방송 연결 URL 입력란 추가 -->
            <div class="flex flex-col sm:flex-row gap-2 max-w-4xl">
                <input type="text" placeholder="실시간 중계방송 또는 구장 IP 카메라 스트림 URL 입력 (예: rtmp://...)" class="flex-1 bg-gray-800 rounded-lg p-3 text-sm text-white border border-gray-700 focus:border-red-500 focus:outline-none shadow-inner">
                <button onclick="showToast('카메라 스트림이 연결되었습니다. 실시간 AI 비전 스캔을 시작합니다.')" class="bg-red-600 hover:bg-red-700 text-white font-bold py-3 px-6 rounded-lg transition whitespace-nowrap shadow-[0_0_15px_rgba(239,68,68,0.4)] flex items-center justify-center">
                    <i data-lucide="video" class="w-4 h-4 mr-2"></i> 카메라 연결
                </button>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-4 gap-6 h-full lg:h-[70vh]">
                <!-- AI 비전 자동 분석 영상 피드 (좌측 3칸) -->
                <div class="lg:col-span-3 flex flex-col space-y-4 h-[40vh] lg:h-full">
                    <div class="bg-black rounded-2xl border-2 border-gray-700 h-full relative overflow-hidden flex items-center justify-center shadow-2xl">
                        <img src="https://images.unsplash.com/photo-1612872087720-bb876e2e67d1?ixlib=rb-4.0.3&auto=format&fit=crop&w=1200&q=80" class="absolute inset-0 w-full h-full object-cover opacity-30">
                        <div class="absolute inset-0 bg-[url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0MCIgaGVpZ2h0PSI0MCI+PHBhdGggZD0iTTAgMGg0MHY0MEgweiIgZmlsbD0ibm9uZSIvPjxwYXRoIGQ0iTTAgMGgxdjQwSDB6TTAgMGg0MHYxSDB6IiBmaWxsPSJyZ2JhKDI1NSwyNTUsMjU1LDAuMSkiLz48L3N2Zz4=')] opacity-50"></div>
                        
                        <div id="ai-tracker-box" class="absolute top-1/3 left-1/2 transform -translate-x-1/2 -translate-y-1/2 border-2 border-green-500 w-32 h-40 sm:w-48 sm:h-56 rounded transition-all duration-1000 flex flex-col justify-end p-2 shadow-[0_0_20px_rgba(34,197,94,0.4)]">
                            <div class="bg-black/70 p-1 rounded">
                                <span id="tracker-player" class="bg-green-500 text-white text-[10px] sm:text-[11px] px-2 py-0.5 rounded font-bold w-max block mb-1">분석 준비</span>
                                <span id="tracker-data" class="text-green-300 text-[9px] sm:text-[10px] font-mono">대기중...</span>
                            </div>
                        </div>

                        <div class="absolute top-4 left-4 bg-red-600 text-white px-3 sm:px-4 py-1.5 sm:py-2 rounded-full text-xs sm:text-sm font-bold flex items-center shadow-lg border border-red-400">
                            <span class="w-2 h-2 sm:w-3 sm:h-3 bg-white rounded-full animate-ping mr-2 sm:mr-3"></span> VISION AUTO-TRACKING
                        </div>
                    </div>
                </div>

                <!-- 실시간 AI 자동 로그 및 벤치 상황판 (우측 1칸) -->
                <div class="glass-panel rounded-2xl p-4 sm:p-5 lg:col-span-1 flex flex-col h-auto min-h-[40vh] lg:h-full overflow-hidden shadow-2xl border border-gray-700">
                    <h2 class="text-base sm:text-lg font-black mb-4 border-b border-gray-700 pb-2 text-white flex items-center">
                        <i data-lucide="zap" class="w-4 h-4 sm:w-5 sm:h-5 mr-2 text-yellow-400"></i> AI 벤치 자동 알림
                    </h2>
                    
                    <!-- 팀 모멘텀(리듬) 게이지 -->
                    <div class="mb-4 sm:mb-6 bg-gray-800/80 p-3 rounded-lg border border-gray-700">
                        <div class="flex justify-between text-[10px] sm:text-xs font-bold mb-2">
                            <span class="text-gray-300">팀 리듬 (모멘텀)</span>
                            <span id="rhythm-text" class="text-green-400">안정적 (100%)</span>
                        </div>
                        <div class="w-full bg-gray-900 rounded-full h-3 sm:h-4 border border-gray-600 overflow-hidden shadow-inner">
                            <div id="rhythm-bar" class="bg-green-500 h-full rounded-full transition-all duration-700" style="width: 100%"></div>
                        </div>
                    </div>

                    <!-- 타임아웃 경고 모달창 -->
                    <div id="timeout-alert" class="hidden bg-red-600 text-white p-3 sm:p-4 rounded-xl mb-4 shadow-[0_0_20px_rgba(239,68,68,0.8)] blink-red border-2 border-white">
                        <div class="flex items-center justify-center mb-1"><i data-lucide="siren" class="w-5 h-5 sm:w-6 sm:h-6 mr-2"></i><span class="font-black text-sm sm:text-lg">작전 타임 즉각 권장!</span></div>
                        <p class="text-[9px] sm:text-[11px] text-center font-bold">어이없는 범실 누적으로 리듬 붕괴.</p>
                    </div>

                    <!-- 실시간 자동 이벤트 로그 -->
                    <div class="flex-1 overflow-hidden flex flex-col">
                        <div class="flex justify-between items-end mb-2">
                            <h3 class="text-[10px] sm:text-xs font-bold text-blue-300 flex items-center"><i data-lucide="activity" class="w-3 h-3 sm:w-4 sm:h-4 mr-1"></i> AI 비전 기록 스트림</h3>
                            <span class="text-[8px] sm:text-[10px] text-gray-500 animate-pulse">수신중...</span>
                        </div>
                        <div class="flex-1 bg-black/50 border border-gray-700 rounded-lg p-2 overflow-y-auto" id="auto-log-container">
                            <ul id="event-log" class="space-y-2 flex flex-col justify-end min-h-full">
                                <!-- JS 자동 추가 -->
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- ==========================================
             [분석관(마스터)] 3. 경기 후 (2분 요약 분석표)
             ========================================== -->
        <section id="view-post" class="hidden space-y-6 max-w-7xl mx-auto">
            <div class="text-center mb-8">
                <h2 class="text-2xl sm:text-3xl font-black text-white mb-2">경기 종합 분석 및 평가서 (Post-Match)</h2>
                <p class="text-xs sm:text-sm text-gray-400">코칭스태프가 2분 안에 파악하고 내일 훈련에 즉시 적용하는 대시보드.</p>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- 레이더 차트 -->
                <div class="glass-panel rounded-2xl p-4 sm:p-6 shadow-xl flex flex-col items-center justify-center">
                    <h3 class="text-base sm:text-lg font-bold mb-4 w-full text-left text-blue-400 flex items-center"><i data-lucide="pie-chart" class="w-4 h-4 sm:w-5 sm:h-5 mr-2"></i> 팀 5대 핵심 지표 분석</h3>
                    <div class="w-full max-w-[250px] sm:max-w-[300px] relative">
                        <canvas id="radarChart"></canvas>
                    </div>
                    <div class="w-full mt-4 bg-gray-800/50 p-3 rounded-lg border border-red-500/30">
                        <p class="text-[10px] sm:text-xs text-red-400 font-bold"><i data-lucide="alert-circle" class="w-3 h-3 inline"></i> 리시브 효율 리그 평균(50%) 미달 (35%).</p>
                    </div>
                </div>

                <!-- 2분 요약 평가서 -->
                <div class="glass-panel rounded-2xl p-4 sm:p-6 lg:col-span-2 shadow-xl flex flex-col">
                    <h3 class="text-base sm:text-lg font-bold mb-4 text-yellow-400 flex items-center"><i data-lucide="file-text" class="w-4 h-4 sm:w-5 sm:h-5 mr-2"></i> 코칭스태프 2분 직관 평가서</h3>
                    
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 flex-1">
                        <!-- 강점 -->
                        <div class="bg-gray-800 border border-green-500/30 rounded-xl p-4 shadow-inner">
                            <h4 class="font-bold text-green-400 mb-2 flex items-center text-sm"><i data-lucide="thumbs-up" class="w-4 h-4 mr-1"></i> 긍정적 요소 (Keep)</h4>
                            <ul class="text-[11px] sm:text-sm text-gray-300 space-y-2 list-disc pl-4 marker:text-green-500">
                                <li><strong>중앙 속공 활용:</strong> 미들블로커 속공 득점률 65% 달성.</li>
                                <li><strong>클러치 집중력:</strong> 20점 이후 범실 단 1개로 멘탈 관리 우수.</li>
                            </ul>
                        </div>
                        
                        <!-- 약점 -->
                        <div class="bg-gray-800 border border-red-500/30 rounded-xl p-4 shadow-inner">
                            <h4 class="font-bold text-red-400 mb-2 flex items-center text-sm"><i data-lucide="thumbs-down" class="w-4 h-4 mr-1"></i> 치명적 약점 (Fix)</h4>
                            <ul class="text-[11px] sm:text-sm text-gray-300 space-y-2 list-disc pl-4 marker:text-red-500">
                                <li><strong>용병 체력 저하:</strong> 3세트 후 스파이크 타점 12cm 하락.</li>
                                <li><strong>플로터 서브 대처:</strong> 목적타 서브에 레프트 리시브 심각한 붕괴.</li>
                            </ul>
                        </div>

                        <!-- 훈련 스케줄 -->
                        <div class="bg-blue-900/30 border border-blue-500/50 rounded-xl p-4 sm:col-span-2 mt-2">
                            <h4 class="font-bold text-blue-400 mb-3 flex items-center text-sm"><i data-lucide="calendar-check" class="w-4 h-4 mr-2"></i> 내일 훈련 자동 스케줄러 (처방전)</h4>
                            <div class="flex flex-col sm:flex-row gap-3">
                                <div class="flex-1 bg-gray-800 rounded p-3 shadow-inner">
                                    <span class="text-[10px] font-bold text-gray-400">용병 전용</span>
                                    <p class="text-[11px] sm:text-xs text-white mt-1">볼 훈련 제외. 하체 코어 및 체력 보강 웨이트 2시간 배정.</p>
                                </div>
                                <div class="flex-1 bg-gray-800 rounded p-3 shadow-inner">
                                    <span class="text-[10px] font-bold text-gray-400">레프트 리시브 라인</span>
                                    <p class="text-[11px] sm:text-xs text-white mt-1">서브 머신 활용 무회전 플로터 서브 200구 집중 리시브 훈련.</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- ==========================================
             [분석관(마스터) 신규 추가] 4. 용병 선발 (Scouting)
             ========================================== -->
        <section id="view-scout" class="hidden space-y-6 max-w-7xl mx-auto">
            <div class="text-center mb-8">
                <h2 class="text-2xl sm:text-3xl font-black text-yellow-500 mb-2">외인 용병 AI 스카우팅 리포트</h2>
                <p class="text-xs sm:text-sm text-gray-400">해외 리그 영상을 AI가 분석하여 타점, 부상 위험도, 클러치 능력을 검증합니다.</p>
            </div>

            <!-- [2번 지시사항 조치] 해외 영상 링크 입력란 추가 -->
            <div class="flex flex-col sm:flex-row gap-2 max-w-4xl mx-auto mb-6">
                <input type="text" placeholder="분석할 해외 리그 선수의 영상 링크(URL) 입력 (예: 유튜브 하이라이트 주소)" class="flex-1 bg-gray-800 rounded-lg p-3 text-sm text-white border border-gray-700 focus:border-yellow-500 focus:outline-none shadow-inner">
                <button onclick="showToast('영상 링크가 인식되었습니다. AI 스카우팅 분석을 추출 중입니다.')" class="bg-yellow-600 hover:bg-yellow-500 text-white font-bold py-3 px-6 rounded-lg transition whitespace-nowrap shadow-[0_0_15px_rgba(202,138,4,0.4)] flex items-center justify-center">
                    <i data-lucide="scan" class="w-4 h-4 mr-2"></i> AI 분석 시작
                </button>
            </div>

            <div class="glass-panel rounded-2xl p-4 sm:p-6 shadow-xl border-t-4 border-yellow-500">
                <div class="flex flex-col md:flex-row gap-6">
                    <!-- 용병 프로필 -->
                    <div class="md:w-1/3 bg-gray-800 rounded-xl p-4 text-center shadow-inner border border-gray-700">
                        <div class="w-24 h-24 mx-auto bg-gray-700 rounded-full mb-4 border-4 border-yellow-500 flex items-center justify-center">
                            <i data-lucide="user" class="w-12 h-12 text-gray-400"></i>
                        </div>
                        <h3 class="text-xl font-black text-white">존 스미스 (John Smith)</h3>
                        <p class="text-sm text-gray-400 mb-4">국적: 미국 / 포지션: OP / 신장: 205cm</p>
                        <div class="bg-gray-900 rounded p-2 text-xs text-left space-y-1">
                            <p><span class="text-gray-500">소속:</span> 이탈리아 A리그</p>
                            <p><span class="text-gray-500">AI 종합평점:</span> <span class="text-yellow-400 font-bold">A- (영입 권장)</span></p>
                        </div>
                    </div>
                    
                    <!-- AI 비전 영상 스캔 결과 -->
                    <div class="md:w-2/3 flex flex-col justify-between">
                        <h4 class="font-bold text-yellow-400 mb-3 flex items-center"><i data-lucide="video" class="w-4 h-4 mr-2"></i> 해외 리그 영상 AI 비전 스캔 결과</h4>
                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                            <div class="bg-gray-800/50 p-4 rounded-lg border border-gray-700">
                                <strong class="text-sm text-gray-300 block mb-1">스파이크 타점 (최대/평균)</strong>
                                <p class="text-lg font-bold text-white">355cm / <span class="text-blue-400">348cm</span></p>
                                <p class="text-[10px] text-gray-500 mt-1">타점 유지력이 V리그 최상위권 수준.</p>
                            </div>
                            <div class="bg-gray-800/50 p-4 rounded-lg border border-gray-700">
                                <strong class="text-sm text-gray-300 block mb-1">부상 위험도 (착지 밸런스)</strong>
                                <p class="text-lg font-bold text-yellow-400">주의 (왼발 쏠림 15%)</p>
                                <p class="text-[10px] text-gray-500 mt-1">스파이크 후 착지 시 왼쪽 무릎 하중 집중됨.</p>
                            </div>
                            <div class="bg-gray-800/50 p-4 rounded-lg border border-gray-700">
                                <strong class="text-sm text-gray-300 block mb-1">클러치 능력 (20점 이후)</strong>
                                <p class="text-lg font-bold text-green-400">공격 성공률 58%</p>
                                <p class="text-[10px] text-gray-500 mt-1">위기 상황 해결 능력이 매우 검증됨.</p>
                            </div>
                            <div class="bg-gray-800/50 p-4 rounded-lg border border-gray-700 flex items-center justify-center">
                                <button onclick="showToast('영입 후보 리스트에 추가되었습니다.')" class="w-full bg-yellow-600 hover:bg-yellow-500 text-white font-bold py-3 rounded transition shadow-lg text-sm">
                                    <i data-lucide="star" class="w-4 h-4 inline mr-1"></i> 영입 후보 등록
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </main>

    <!-- 커스텀 토스트 알림창 (팝업 차단 회피) -->
    <div id="toast" class="fixed bottom-10 left-1/2 transform -translate-x-1/2 bg-gray-800 text-white px-6 py-3 rounded-full opacity-0 transition-opacity duration-300 pointer-events-none z-[300] text-xs sm:text-sm shadow-xl font-bold text-center w-max border border-gray-600">
        메시지
    </div>

    <script>
        lucide.createIcons();

        // 토스트 UI
        function showToast(msg) {
            const toast = document.getElementById('toast');
            toast.innerText = msg;
            toast.classList.remove('opacity-0');
            setTimeout(() => { toast.classList.add('opacity-0'); }, 3000);
        }

        /* ==========================================
           접근 권한 관리 (RBAC) 로직
           ========================================== */
        let currentRole = null;

        function login(role) {
            currentRole = role;
            document.getElementById('login-screen').classList.add('hidden'); // 로그인 창 숨김
            
            const badge = document.getElementById('role-badge');
            const navTabs = document.getElementById('nav-tabs');
            const mobileNav = document.getElementById('mobile-nav');
            navTabs.classList.remove('hidden');
            mobileNav.classList.remove('hidden');

            // 모든 탭 버튼 숨기기 초기화
            document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.add('hidden'));

            if(role === 'PLAYER') {
                badge.innerText = "선수 접속";
                badge.className = "bg-gray-700 px-2 py-0.5 rounded text-[10px] font-bold text-white ml-2 border border-gray-600";
                document.getElementById('tab-pre-player').classList.remove('hidden');
                setupMobileNav('PLAYER');
                switchTab('pre-player');
            } 
            else if(role === 'COACH') {
                badge.innerText = "코칭스태프 접속";
                badge.className = "bg-green-800 px-2 py-0.5 rounded text-[10px] font-bold text-white ml-2 border border-green-600";
                document.getElementById('tab-pre-coach').classList.remove('hidden');
                setupMobileNav('COACH');
                switchTab('pre-coach');
            } 
            else if(role === 'ANALYST') {
                badge.innerText = "마스터 (분석관)";
                badge.className = "bg-blue-700 px-2 py-0.5 rounded text-[10px] font-bold text-white ml-2 border border-blue-400 shadow-[0_0_10px_rgba(59,130,246,0.8)]";
                
                // 마스터는 전체 메뉴 노출
                document.getElementById('tab-pre-master').classList.remove('hidden');
                document.getElementById('tab-in').classList.remove('hidden');
                document.getElementById('tab-post').classList.remove('hidden');
                document.getElementById('tab-scout').classList.remove('hidden'); // 용병 선발 추가
                setupMobileNav('ANALYST');
                switchTab('pre-master');
            }
        }

        function logout() {
            currentRole = null;
            // 모든 뷰 숨기기
            ['pre-player', 'pre-coach', 'pre-master', 'in', 'post', 'scout'].forEach(id => {
                document.getElementById('view-' + id).classList.add('hidden');
                document.getElementById('view-' + id).classList.remove('block');
            });
            document.getElementById('nav-tabs').classList.add('hidden');
            document.getElementById('mobile-nav').classList.add('hidden');
            document.getElementById('login-screen').classList.remove('hidden');
            
            // 시뮬레이션 정지
            if(window.autoSimInterval) { clearInterval(window.autoSimInterval); window.autoSimStarted = false; }
        }

        function setupMobileNav(role) {
            const mNav = document.getElementById('mobile-nav');
            if(role === 'PLAYER') {
                mNav.innerHTML = `<button class="text-blue-500 flex flex-col items-center"><i data-lucide="clipboard-edit" class="w-5 h-5 mb-1"></i><span class="text-[10px] font-bold">자가진단</span></button>`;
            } else if(role === 'COACH') {
                mNav.innerHTML = `<button class="text-green-500 flex flex-col items-center"><i data-lucide="users" class="w-5 h-5 mb-1"></i><span class="text-[10px] font-bold">선수평가</span></button>`;
            } else if(role === 'ANALYST') {
                mNav.innerHTML = `
                    <button onclick="switchTab('pre-master')" class="text-gray-400 hover:text-white flex flex-col items-center"><i data-lucide="clipboard-list" class="w-5 h-5 mb-1"></i><span class="text-[10px]">1.경기전</span></button>
                    <button onclick="switchTab('in')" class="text-gray-400 hover:text-white flex flex-col items-center"><i data-lucide="radio" class="w-5 h-5 mb-1"></i><span class="text-[10px]">2.실시간</span></button>
                    <button onclick="switchTab('post')" class="text-gray-400 hover:text-white flex flex-col items-center"><i data-lucide="bar-chart-2" class="w-5 h-5 mb-1"></i><span class="text-[10px]">3.분석서</span></button>
                    <button onclick="switchTab('scout')" class="text-yellow-600 hover:text-yellow-400 flex flex-col items-center"><i data-lucide="star" class="w-5 h-5 mb-1"></i><span class="text-[10px]">4.용병</span></button>
                `;
            }
            lucide.createIcons();
        }

        // 탭 전환 로직
        function switchTab(tabId) {
            // 모든 뷰 끄기
            ['pre-player', 'pre-coach', 'pre-master', 'in', 'post', 'scout'].forEach(id => {
                const view = document.getElementById('view-' + id);
                if(view) { view.classList.add('hidden'); view.classList.remove('block', 'animate-fade-in'); }
                const tab = document.getElementById('tab-' + id);
                if(tab) tab.classList.remove('active', 'border-blue-500', 'text-white', 'border-yellow-500', 'text-yellow-400');
            });
            
            // 선택된 뷰 켜기
            document.getElementById('view-' + tabId).classList.remove('hidden');
            document.getElementById('view-' + tabId).classList.add('block', 'animate-fade-in');
            
            const activeTab = document.getElementById('tab-' + tabId);
            if(activeTab) {
                if(tabId === 'scout') activeTab.classList.add('active', 'border-yellow-500', 'text-yellow-400');
                else activeTab.classList.add('active', 'border-blue-500', 'text-white');
            }
            
            // 차트 및 시뮬레이션
            if(tabId === 'post' && !window.radarChartCreated) { initRadarChart(); window.radarChartCreated = true; }
            if(tabId === 'in' && !window.autoSimStarted) { startAIAutoSimulation(); window.autoSimStarted = true; }
        }

        // 시계
        setInterval(() => {
            const now = new Date();
            document.getElementById('clock').innerText = now.toLocaleTimeString('en-US', { hour: '2-digit', minute:'2-digit', hour12: false });
        }, 1000);

        /* ==============================================================
           경기 중 완전 자동화(AI Vision Auto-Tracking) 시뮬레이션
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
            window.autoSimInterval = setInterval(() => {
                if(document.getElementById('view-in').classList.contains('hidden')) return;
                
                const evt = autoEvents[eventCount % autoEvents.length];
                
                const tracker = document.getElementById('ai-tracker-box');
                const tPlayer = document.getElementById('tracker-player');
                const tData = document.getElementById('tracker-data');
                
                if(tracker) {
                    tracker.style.left = evt.tracker.x;
                    tracker.style.top = evt.tracker.y;
                    tracker.className = `absolute transform -translate-x-1/2 -translate-y-1/2 border-2 w-32 h-40 sm:w-48 sm:h-56 rounded transition-all duration-1000 flex flex-col justify-end p-2 border-${evt.color}-500 shadow-[0_0_20px_rgba(0,0,0,0.8)] z-10 bg-${evt.color}-900/20`;
                    
                    tPlayer.className = `bg-${evt.color}-600 text-white text-[10px] sm:text-[11px] px-2 py-0.5 rounded font-bold w-max block mb-1`;
                    tPlayer.innerText = evt.tracker.text;
                    tData.innerText = evt.detail;
                    tData.className = `text-${evt.color}-300 text-[9px] sm:text-[10px] font-bold`;
                }

                addAutoLog(evt.action, evt.detail, evt.border);
                updateAutoRhythm(evt.rhythm);
                eventCount++;
            }, 3500);
        }

        function addAutoLog(action, detail, borderClass) {
            const logUl = document.getElementById('event-log');
            if(!logUl) return;
            const newLi = document.createElement('li');
            newLi.className = `text-[10px] sm:text-xs text-white bg-gray-800 p-2 sm:p-2.5 rounded-lg flex flex-col border-l-4 ${borderClass} animate-fade-in shadow-md`;
            
            const timeStr = new Date().toLocaleTimeString('en-US', { hour12: false });
            newLi.innerHTML = `
                <div class="flex items-center">
                    <span class="text-gray-400 font-mono w-12 sm:w-16 text-[8px] sm:text-[10px]">${timeStr}</span> 
                    <span class="font-bold text-[11px] sm:text-[13px] ml-1">${action}</span>
                </div>
                <span class="text-[8px] sm:text-[10px] text-gray-400 mt-1 ml-13 sm:ml-17">${detail}</span>
            `;
            logUl.appendChild(newLi);
            
            const container = document.getElementById('auto-log-container');
            container.scrollTop = container.scrollHeight;
            if(logUl.children.length > 8) logUl.removeChild(logUl.firstChild);
        }

        function updateAutoRhythm(amount) {
            rhythmScore += amount;
            if(rhythmScore > 100) rhythmScore = 100;
            if(rhythmScore < 0) rhythmScore = 0;

            const bar = document.getElementById('rhythm-bar');
            const text = document.getElementById('rhythm-text');
            const alertBox = document.getElementById('timeout-alert');
            if(!bar) return;

            bar.style.width = rhythmScore + '%';
            
            if(rhythmScore <= 40) {
                bar.className = "bg-red-500 h-full rounded-full transition-all duration-300";
                text.className = "text-red-400 blink-red font-black";
                text.innerText = `위험! 붕괴 중 (${rhythmScore}%)`;
                alertBox.classList.remove('hidden');
            } else if(rhythmScore <= 70) {
                bar.className = "bg-yellow-500 h-full rounded-full transition-all duration-300";
                text.className = "text-yellow-400 font-bold";
                text.innerText = `주의 요망 (${rhythmScore}%)`;
                alertBox.classList.add('hidden');
            } else {
                bar.className = "bg-green-500 h-full rounded-full transition-all duration-300";
                text.className = "text-green-400 font-bold";
                text.innerText = `안정적 (${rhythmScore}%)`;
                alertBox.classList.add('hidden');
            }
        }

        function initRadarChart() {
            const canvas = document.getElementById('radarChart');
            if(!canvas) return;
            const ctx = canvas.getContext('2d');
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
                            pointLabels: { color: '#cbd5e1', font: { size: 9, sm: {size: 11}, weight: 'bold' } },
                            ticks: { display: false, min: 0, max: 100 }
                        }
                    },
                    plugins: { legend: { position: 'bottom', labels: { color: '#cbd5e1', font: {size: 10} } } }
                }
            });
        }
    </script>
</body>
</html>
