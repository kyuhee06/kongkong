<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>콩 심은 데 콩 난다</title>
    <style>
        /* 기본 스타일 및 무한 스크롤(넓은 흙밭) 느낌 구현 */
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: auto;
            background-color: #9cb945; /* 시안의 초록/흙빛 배경색 */
            font-family: 'Galmuri11', 'Clear Sans', sans-serif;
            user-select: none;
        }

        /* 3000x3000px 크기의 거대한 밭 */
        #field {
            position: relative;
            width: 3000px;
            height: 3000px;
            cursor: dashed;
        }

        /* 고정 UI: 타이틀 및 안내문구 */
        #ui-header {
            position: fixed;
            top: 20px;
            left: 20px;
            color: white;
            font-size: 24px;
            font-weight: bold;
            text-shadow: 2px 2px 0px #000;
            z-index: 10;
        }
        #ui-instruction {
            position: fixed;
            top: 20px;
            right: 20px;
            color: white;
            text-align: right;
            text-shadow: 1px 1px 0px #000;
            z-index: 10;
            font-size: 14px;
            line-height: 1.6;
        }
        #history-list {
            margin-top: 10px;
            color: #d1ff36;
            font-weight: bold;
            list-style: none;
            padding: 0;
        }

        /* 밭(Plot) 요소 스타일 */
        .plot-container {
            position: absolute;
            display: flex;
            flex-direction: column;
            align-items: center;
            transform: translate(-50%, -50%);
            transition: transform 0.2s;
        }

        /* 갈아엎은 땅 */
        .patch {
            width: 120px;
            height: 70px;
            background-color: #e3b265;
            border-radius: 50%;
            box-shadow: inset 0 5px 5px rgba(0,0,0,0.2);
            position: relative;
            display: flex;
            justify-content: center;
            align-items: center;
            border: 2px dashed #c59448;
        }
        .patch.planted {
            border: none;
            background: none;
            box-shadow: none;
        }

        /* 심어진 이미지 */
        .planted-image {
            max-width: 150px;
            max-height: 120px;
            object-fit: contain;
            border-radius: 4px;
            box-shadow: 3px 3px 10px rgba(0,0,0,0.3);
            cursor: pointer;
            z-index: 2;
        }

        /* 푯말 */
        .signpost {
            margin-top: 5px;
            background-color: #c9843b;
            color: white;
            padding: 4px 12px;
            border-radius: 4px;
            font-size: 12px;
            border: 2px solid #a06222;
            box-shadow: 1px 2px 3px rgba(0,0,0,0.3);
            cursor: pointer;
            z-index: 3;
        }

        /* 모달 스타일 */
        .modal {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 320px;
            background-color: #ca934a;
            border: 4px solid #8d5e27;
            border-radius: 20px;
            padding: 20px;
            box-shadow: 5px 5px 0px rgba(0,0,0,0.2);
            z-index: 100;
            color: white;
            text-align: center;
        }
        .modal-close {
            position: absolute;
            top: 10px;
            right: 15px;
            cursor: pointer;
            font-size: 20px;
            color: #724617;
        }
        .modal input[type="text"] {
            width: 80%;
            padding: 8px;
            margin: 15px 0;
            border: 2px solid #8d5e27;
            border-radius: 8px;
            background-color: #f7e0bc;
            color: #5c3a14;
            font-weight: bold;
            text-align: center;
        }
        .modal button {
            background-color: #8d5e27;
            color: white;
            border: none;
            padding: 8px 20px;
            border-radius: 10px;
            cursor: pointer;
            font-weight: bold;
        }
        .modal button:hover {
            background-color: #724617;
        }
        .modal-overlay {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.3);
            z-index: 99;
        }
    </style>
</head>
<body>

    <div id="ui-header">콩 심은 데 🫘 난다</div>
    <div id="ui-instruction">
        원하는 곳을 클릭하고 텃밭을 만들어 보세요.<br>
        비어있는 텃밭에 사진을 심어보세요.
        <ul id="history-list">
            </ul>
    </div>

    <div id="field"></div>

    <div class="modal-overlay" id="overlay"></div>

    <div class="modal" id="modal-till">
        <div class="modal-close" onclick="closeAllModals()">×</div>
        <p style="margin:0; font-weight:bold;">어떤 밭을 갈까요?</p>
        <input type="text" id="input-keyword" placeholder="예: 비행기, 고양이" maxlength="10">
        <br>
        <button onclick="createPlot()">완료</button>
    </div>

    <div class="modal" id="modal-plant">
        <div class="modal-close" onclick="closeAllModals()">×</div>
        <p style="margin:0; font-weight:bold;"><span id="target-keyword" style="color:#fff3cc;"></span> 심을 데</p>
        <p style="font-size:12px; margin:5px 0;">이미지를 선택해주세요.</p>
        <input type="file" id="input-file" accept="image/*" style="margin: 10px 0;">
        <br>
        <button onclick="plantImage()">심기</button>
    </div>

    <div class="modal" id="modal-harvest">
        <div class="modal-close" onclick="closeAllModals()">×</div>
        <p style="margin:0; font-weight:bold;"><span id="harvest-keyword" style="color:#fff3cc;"></span> 심은 데</p>
        <input type="text" id="input-result" placeholder="무엇이 났나요?" maxlength="10"> <span style="font-weight:bold;">났다!</span>
        <p style="font-size:11px; color:#fcecd2;">수확하면 이미지가 저장되고 밭이 비워집니다.</p>
        <button onclick="harvestPlot()">수확 완료</button>
    </div>

    <script>
        const field = document.getElementById('field');
        const overlay = document.getElementById('overlay');
        
        // 현재 작업 중인 좌표 및 상태 저장용 임시 변수
        let currentClickX = 0;
        let currentClickY = 0;
        let activePlotId = null;

        // 전체 밭 데이터 관리 (메모리 저장 - 새로고침하면 초기화됨)
        let plotsData = {};

        // 화면 중앙으로 스크롤 시켜두기 (초기 진입 시 흙밭 한가운데로)
        window.scrollTo(1200, 1200);

        // 빈 땅 클릭 시 -> 밭 갈기 모달 띄우기
        field.addEventListener('click', (e) => {
            if (e.target === field) {
                currentClickX = e.pageX;
                currentClickY = e.pageY;
                
                openModal('modal-till');
            }
        });

        function openModal(id) {
            overlay.style.display = 'block';
            document.getElementById(id).style.display = 'block';
        }

        function closeAllModals() {
            overlay.style.display = 'none';
            document.querySelectorAll('.modal').forEach(m => m.style.display = 'none');
            // 입력창 초기화
            document.getElementById('input-keyword').value = '';
            document.getElementById('input-file').value = '';
            document.getElementById('input-result').value = '';
        }

        // 1. 밭 갈기 실행
        function createPlot() {
            const keyword = document.getElementById('input-keyword').value.trim();
            if (!keyword) return alert('키워드를 입력해주세요!');

            const id = 'plot_' + Date.now();
            
            // 데이터 생성
            plotsData[id] = {
                id: id,
                x: currentClickX,
                y: currentClickY,
                keyword: keyword,
                status: 'till', // till -> planted
                imageSrc: null
            };

            // DOM 생성
            renderPlot(plotsData[id]);
            closeAllModals();
        }

        // 밭 렌더링 함수
        function renderPlot(data) {
            // 이미 존재하는 엘리먼트가 있다면 지우고 새로 그림 (업데이트 대응)
            const existing = document.getElementById(data.id);
            if (existing) existing.remove();

            const container = document.createElement('div');
            container.className = 'plot-container';
            container.id = data.id;
            container.style.left = data.x + 'px';
            container.style.top = data.y + 'px';

            if (data.status === 'till') {
                // 갈아놓은 상태
                const patch = document.createElement('div');
                patch.className = 'patch';
                patch.addEventListener('click', () => triggerPlant(data.id));
                container.appendChild(patch);
            } else if (data.status === 'planted') {
                // 이미지가 심어진 상태
                const img = document.createElement('img');
                img.src = data.imageSrc;
                img.className = 'planted-image';
                img.addEventListener('click', () => triggerHarvest(data.id));
                container.appendChild(img);
            }

            // 푯말은 항상 붙음
            const sign = document.createElement('div');
            sign.className = 'signpost';
            sign.innerText = data.keyword;
            sign.addEventListener('click', (e) => {
                e.stopPropagation(); // 컨테이너나 패치 클릭 이벤트 중복 방지
                if (data.status === 'till') triggerPlant(data.id);
                else triggerHarvest(data.id);
            });
            container.appendChild(sign);

            field.appendChild(container);
        }

        // 2. 심기 모달 트리거
        function triggerPlant(id) {
            activePlotId = id;
            document.getElementById('target-keyword').innerText = `'{${plotsData[id].keyword}}'`;
            openModal('modal-plant');
        }

        // 2. 이미지 심기 실행
        function plantImage() {
            const fileInput = document.getElementById('input-file');
            if (!fileInput.files || !fileInput.files[0]) return alert('이미지 파일을 선택해주세요!');

            const file = fileInput.files[0];
            const reader = new FileReader();

            reader.onload = function(e) {
                plotsData[activePlotId].status = 'planted';
                plotsData[activePlotId].imageSrc = e.target.result; // Base64 이미지 데이터
                
                renderPlot(plotsData[activePlotId]);
                closeAllModals();
            };
            reader.readAsDataURL(file);
        }

        // 3. 수확 모달 트리거
        function triggerHarvest(id) {
            activePlotId = id;
            document.getElementById('harvest-keyword').innerText = `'{${plotsData[id].keyword}}'`;
            openModal('modal-harvest');
        }

        // 3. 수확 및 문장 완성 실행
        function harvestPlot() {
            const resultInput = document.getElementById('input-result').value.trim();
            if (!resultInput) return alert('무엇이 났는지 입력해주세요!');

            const data = plotsData[activePlotId];

            // 가상의 이미지 다운로드 기능 (실제 파일 다운로드 트리거)
            const a = document.createElement('a');
            a.href = data.imageSrc;
            a.download = `${data.keyword}_심은데_${resultInput}_났다.png`;
            a.click();

            // 우측 상단 전광판/히스토리에 문장 추가
            const historyList = document.getElementById('history-list');
            const li = document.createElement('li');
            li.innerText = `✨ ${data.keyword} 심은 데 ${resultInput} 났다!`;
            // 최신 글이 위로 오도록 추가
            historyList.insertBefore(li, historyList.firstChild);

            // 밭 비우기 (데이터 삭제 및 화면에서 제거)
            document.getElementById(activePlotId).remove();
            delete plotsData[activePlotId];

            closeAllModals();
        }
    </script>
</body>
</html>