<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>티켓팅 연습 사이트</title>
    <style>
        :root {
            --main-pink: #f7a7bb; /* 연핑크 */
            --soft-pink: #fce4ec; /* 아주 연한 핑크 */
            --point-pink: #e91e63; /* 찐핑크 (강조색) */
            --text-gray: #666;
        }

        body {
            font-family: 'Pretendard', -apple-system, sans-serif;
            margin: 0;
            background-color: var(--soft-pink);
            display: flex;
            justify-content: center;
            color: #333;
        }

        #app-container {
            width: 100%;
            max-width: 450px;
            min-height: 100vh;
            background: white;
            box-shadow: 0 0 20px rgba(0,0,0,0.05);
            display: flex;
            flex-direction: column;
            position: relative;
        }

        .content-card {
            margin: 20px;
            border: 1px solid #eee;
            border-radius: 30px;
            overflow: hidden;
            flex-grow: 1;
            display: flex;
            flex-direction: column;
        }

        .card-top-bar {
            background-color: var(--main-pink);
            color: white;
            text-align: center;
            padding: 12px;
            font-weight: bold;
            font-size: 16px;
        }

        .section { display: none; padding: 20px; flex-grow: 1; animation: fadeIn 0.3s; }
        .section.active { display: flex; flex-direction: column; }

        /* 1. 안내 문구: 한 줄 정렬 */
        .info-box {
            border: 2px dashed var(--main-pink);
            border-radius: 20px;
            padding: 20px;
            margin-top: 20px;
            text-align: center;
        }
        .info-line { 
            font-size: 13px; 
            color: var(--point-pink); /* 찐핑크로 통일 */
            line-height: 1.8; 
            word-break: keep-all;
            display: block; /* 한 줄씩 정렬 */
        }

        /* 2. 타이머 및 버튼 */
        .timer-circle {
            width: 150px;
            height: 150px;
            border: 4px solid var(--soft-pink);
            border-radius: 50%;
            margin: 30px auto;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        .timer-value { font-size: 22px; font-weight: bold; color: var(--point-pink); }

        .btn-main {
            background-color: var(--main-pink);
            color: white;
            border: none;
            padding: 15px;
            border-radius: 25px;
            font-size: 18px;
            font-weight: bold;
            cursor: pointer;
            width: 100%;
            margin-top: 10px;
        }
        .btn-main:disabled { background-color: #eee; color: #ccc; cursor: not-allowed; }

        /* 3. 대기 화면: 하트 제거 및 깔끔한 UI */
        .waiting-content { text-align: center; padding-top: 100px; }
        .waiting-msg { font-size: 20px; font-weight: bold; color: var(--point-pink); margin-bottom: 15px; }

        /* 4. 보안 문자 디자인: 연핑크 배경, 찐핑크 텍스트, 직선 폰트 */
        .captcha-box { 
            background: var(--soft-pink); 
            padding: 25px; 
            font-size: 36px; 
            font-weight: 800; 
            letter-spacing: 8px; 
            margin: 20px 0; 
            text-align: center; 
            border-radius: 15px; 
            color: var(--point-pink);
            border: 2px solid var(--main-pink);
            font-style: normal; /* 기울임 제거 */
        }

        /* 5. 좌석 구역 색상 수정 */
        .area-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
        /* A-D: 찐핑크 */
        .area-btn.heavy { background-color: var(--point-pink); color: white; border: none; }
        /* 101-104: 흰색 배경 + 핑크 테두리 */
        .area-btn.light { background-color: white; color: var(--point-pink); border: 2px solid var(--main-pink); }
        .area-btn { padding: 15px 5px; border-radius: 10px; text-align: center; cursor: pointer; font-weight: bold; font-size: 13px; }

        .seat-grid { display: grid; grid-template-columns: repeat(10, 1fr); gap: 5px; justify-content: center; }
        .seat { width: 25px; height: 25px; background: #eee; border-radius: 4px; cursor: pointer; transition: 0.1s; }
        .seat.podo { background: #8a2be2; box-shadow: 0 0 5px rgba(138, 43, 226, 0.4); }

        /* 6. 최종 결과 화면: 텍스트 강조 */
        .modal { 
            display: none; position: absolute; top:0; left:0; width:100%; height:100%; 
            background:white; z-index:2000; flex-direction: column;
            justify-content:center; align-items:center; text-align:center; padding: 20px; 
        }
        .success-title { 
            font-size: 32px; /* 크기 증가 */
            font-weight: 900; /* 두께 강화 */
            color: var(--point-pink); 
            margin-bottom: 10px;
        }

        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    </style>
</head>
<body>

<div id="app-container">
    <div class="content-card">
        <div class="card-top-bar" id="card-title">예매하기</div>

        <div id="sec-main" class="section active">
            <div class="info-box">
                <span class="info-line">안심예매 단계로 보안 문자를 입력해야 합니다.</span>
                <span class="info-line">정각 00초에 ‘예매하기’ 버튼이 활성화됩니다.</span>
            </div>
            <div class="timer-circle">
                <span style="font-size:12px; color:var(--point-pink); font-weight:bold;">서버 시간</span>
                <span class="timer-value" id="server-time">00:00:00</span>
            </div>
            <button class="btn-main" id="booking-btn" onclick="startBooking()" disabled>예매하기</button>
        </div>

        <div id="sec-waiting" class="section">
            <div class="waiting-content">
                <div class="waiting-msg">잠시만 기다려 주세요!</div>
                <div style="font-size: 16px; color: #555;">고객님 앞에 <span id="wait-num" style="color:var(--point-pink); font-weight:bold;">0</span>명이 있어요.</div>
            </div>
        </div>

        <div id="sec-captcha" class="section">
            <h3 style="text-align:center; color:var(--point-pink);">보안 문자 입력</h3>
            <div class="captcha-box" id="captcha-code">A7B2C</div>
            <input type="text" style="width:100%; padding:15px; border:2px solid var(--soft-pink); border-radius:10px; text-align:center; font-size:18px; color:var(--point-pink);" id="captcha-input" maxlength="5" oninput="this.value = this.value.toUpperCase()">
            <button class="btn-main" style="margin-top:auto" onclick="verifyCaptcha()">입력 완료</button>
        </div>

        <div id="sec-area" class="section">
            <div style="background:var(--soft-pink); padding:10px; text-align:center; margin-bottom:20px; font-weight:bold; color:var(--point-pink); border-radius:10px;">STAGE</div>
            <div class="area-grid" id="area-grid"></div>
        </div>

        <div id="sec-seat" class="section">
            <div style="text-align:right; margin-bottom:10px; font-size:12px; color:var(--point-pink); font-weight:bold;">남은 좌석: <span id="remain-txt">0</span></div>
            <div class="seat-grid" id="seat-grid"></div>
        </div>

        <div id="sec-payment" class="section">
            <div style="background:var(--soft-pink); padding:20px; border-radius:15px; margin-bottom:20px; color:var(--point-pink);">
                <p>선택 좌석: <strong id="final-seat">B구역 16번</strong></p>
                <p>총 결제 금액: <strong>167,000원</strong></p>
            </div>
            <div style="display:grid; grid-template-columns:1fr 1fr; gap:10px; margin-bottom:20px;">
                <div class="area-btn light" onclick="selectPay('card', this)">신용 카드</div>
                <div class="area-btn light" onclick="selectPay('bank', this)">무통장 입금</div>
            </div>
            <button class="btn-main" id="final-pay-btn" onclick="completeBooking()" disabled>최종 결제 완료</button>
        </div>
    </div>

    <div id="final-modal" class="modal">
        <div class="success-title" id="success-title-text">🎉 예매 성공!</div>
        <div id="success-msg" style="font-size: 16px; color: #666; margin-bottom:30px;"></div>
        <button class="btn-main" style="max-width: 200px;" onclick="location.reload()">처음으로 돌아가기</button>
    </div>
</div>

<script>
    let selectedPayMethod = "";
    let seatInterval;

    // 서버 시간 및 버튼 활성화 (정각 00초)
    function updateTime() {
        const now = new Date();
        const s = now.getSeconds();
        const timeStr = now.toTimeString().split(' ')[0];
        document.getElementById('server-time').innerText = timeStr;
        
        const btn = document.getElementById('booking-btn');
        btn.disabled = (s !== 0); // 0초일 때만 false(활성화)
        requestAnimationFrame(updateTime);
    }
    updateTime();

    function showSection(id) {
        document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
        document.getElementById(id).classList.add('active');
    }

    // 대기열 로직
    function startBooking() {
        showSection('sec-waiting');
        let count = Math.floor(Math.random() * 20000) + 15000;
        const timer = setInterval(() => {
            count -= Math.floor(Math.random() * 800) + 300;
            if(count <= 0) {
                clearInterval(timer);
                generateCaptcha();
                showSection('sec-captcha');
            }
            document.getElementById('wait-num').innerText = count.toLocaleString();
        }, 80);
    }

    function generateCaptcha() {
        const chars = "ABCDEFGHJKLMNPQRSTUVWXYZ23456789";
        let code = "";
        for(let i=0; i<5; i++) code += chars.charAt(Math.floor(Math.random()*chars.length));
        document.getElementById('captcha-code').innerText = code;
    }

    function verifyCaptcha() {
        const input = document.getElementById('captcha-input').value;
        const code = document.getElementById('captcha-code').innerText;
        if(input === code) {
            renderAreas();
            showSection('sec-area');
        } else {
            alert('보안 문자가 틀렸습니다!');
            generateCaptcha();
            document.getElementById('captcha-input').value = "";
        }
    }

    function renderAreas() {
        const grid = document.getElementById('area-grid');
        grid.innerHTML = '';
        const heavy = ['A','B','C','D'];
        const light = ['101','102','103','104'];
        heavy.forEach(a => grid.innerHTML += `<div class="area-btn heavy" onclick="initSeats('${a}')">${a} 구역</div>`);
        light.forEach(a => grid.innerHTML += `<div class="area-btn light" onclick="initSeats('${a}')">${a} 구역</div>`);
    }

    // 좌석 선택 로직 (난이도 강화)
    function initSeats(name) {
        showSection('sec-seat');
        if(seatInterval) clearInterval(seatInterval);
        refreshSeats(name);
        seatInterval = setInterval(() => refreshSeats(name), 1200); // 소량 생성-제거 반복
    }

    function refreshSeats(areaName) {
        const grid = document.getElementById('seat-grid');
        grid.innerHTML = '';
        let podoCount = 0;

        for(let i=1; i<=100; i++) {
            const s = document.createElement('div');
            s.className = 'seat';
            
            // 난이도 고증: 앞자리(1-50)는 확률 매우 낮음(2%), 뒷자리는 10%
            let prob = i <= 50 ? 0.98 : 0.90;
            if(Math.random() > prob) {
                s.classList.add('podo');
                podoCount++;
            }

            s.onclick = () => {
                if(s.classList.contains('podo')) {
                    // 이선좌 확률 90%
                    if(Math.random() < 0.90) {
                        alert('이미 선택된 좌석입니다.');
                        s.classList.remove('podo');
                    } else {
                        clearInterval(seatInterval);
                        document.getElementById('final-seat').innerText = areaName + '구역 ' + i + '번';
                        showSection('sec-payment');
                    }
                }
            };
            grid.appendChild(s);
        }
        document.getElementById('remain-txt').innerText = podoCount;
    }

    function selectPay(method, el) {
        selectedPayMethod = method;
        document.querySelectorAll('#sec-payment .area-btn').forEach(b => {
            b.style.border = "2px solid var(--main-pink)";
            b.style.backgroundColor = "white";
        });
        el.style.backgroundColor = var(--soft-pink);
        el.style.border = "2px solid var(--point-pink)";
        document.getElementById('final-pay-btn').disabled = false;
    }

    function completeBooking() {
        const modal = document.getElementById('final-modal');
        const msgDiv = document.getElementById('success-msg');
        
        let text = (selectedPayMethod === 'card') 
            ? "🎉 예매 성공! 카드 승인이 완료되었습니다!" 
            : "🎉 예매 성공! 내일까지 입금하지 않으면 자동 취소되니 주의하세요!";
        
        msgDiv.innerText = text;
        modal.style.display = 'flex';
    }
</script>
</body>
