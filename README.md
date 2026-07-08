<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>Калькулятор расходных материалов</title>
    <style type="text/css">
        :root {
            --primary: #0d47a1;
            --primary-light: #e3f2fd;
            --accent: #d32f2f;
            --success: #2e7d32;
            --text-main: #2c3e50;
            --text-muted: #636e72;
            --bg-body: #f0f2f5;
            --white: #ffffff;
            --border: #dcdde1;
            --radius: 10px;
            --shadow: 0 10px 25px rgba(0,0,0,0.05);
        }
        body {
            font-family: 'Segoe UI', Roboto, sans-serif;
            background-color: var(--bg-body);
            color: var(--text-main);
            margin: 0;
            padding: 20px;
            line-height: 1.6;
        }
        .main-container {
            max-width: 1100px;
            margin: 0 auto;
            background: var(--white);
            padding: 40px;
            border-radius: var(--radius);
            box-shadow: var(--shadow);
            position: relative;
        }
        header {
            text-align: center;
            border-bottom: 3px solid var(--primary);
            padding-bottom: 20px;
            margin-bottom: 35px;
        }
        header h1 { color: var(--primary); font-size: 28px; margin: 0 0 5px; text-transform: uppercase; }
        header p { color: var(--accent); font-weight: 800; margin: 0; font-size: 14px; }
        .catalog-manager {
            background: var(--primary-light);
            padding: 20px;
            border-radius: var(--radius);
            margin-bottom: 30px;
            border: 1px solid var(--border);
        }
        .manager-grid {
            display: grid;
            grid-template-columns: 2fr 1fr 1fr auto auto;
            gap: 10px;
            align-items: flex-end;
        }
        .table-wrapper { 
            overflow: visible; 
            margin-bottom: 20px;
            height: auto;
            max-height: none;
        }
        .calc-table { width: 100%; border-collapse: collapse; }
        .calc-table th {
            background-color: #f8f9fa;
            color: var(--primary);
            text-align: left;
            padding: 15px;
            font-size: 13px;
            border-bottom: 2px solid var(--border);
        }
        .calc-table td { padding: 12px 10px; border-bottom: 1px solid #eee; position: relative; }
        input {
            width: 100%; 
            border-collapse: collapse; 
            overflow: visible;
        }
        .autocomplete-container { position: relative; width: 100%; }
        .suggestions-list {
            position: absolute;
            top: 100%;
            left: 0;
            right: 0;
            background: white;
            border: 1px solid var(--primary);
            z-index: 9999;
            max-height: 300px;
            overflow-y: auto;
            box-shadow: 0 8px 16px rgba(0,0,0,0.2);
            display: none;
            border-radius: 0 0 6px 6px;
        }
        .suggestion-item {
            padding: 12px 15px;
            cursor: pointer;
            font-size: 13px;
            border-bottom: 1px solid #f0f0f0;
            white-space: normal; 
            line-height: 1.4;
            color: #2d3436;
        }
        .suggestion-item:hover { background-color: #f1f7ff; color: var(--primary); font-weight: 500; }
        .unit-cell { color: var(--text-muted); font-size: 13px; text-align: center; }
        .sum-cell { color: var(--accent); font-weight: 700; text-align: right; font-size: 16px; }
        .btn {
            cursor: pointer;
            border: none;
            border-radius: 6px;
            font-weight: 600;
            padding: 12px 20px;
            display: inline-flex;
            align-items: center;
            gap: 8px;
            transition: 0.2s;
        }
        .btn-add-row { background-color: var(--primary); color: white; margin-bottom: 20px; }
        .btn-save { background-color: var(--success); color: white; }
        .btn-delete { background: none; color: #b2bec3; font-size: 18px; cursor: pointer; }
        .btn-delete:hover { color: var(--accent); }
        .sticky-footer-panel {
            position: sticky;
            bottom: 20px;
            background: var(--text-main);
            color: white;
            padding: 20px;
            border-radius: var(--radius);
            box-shadow: 0 -5px 25px rgba(0,0,0,0.15);
            margin-top: 40px;
            z-index: 90;
        }
        .footer-flex { display: flex; justify-content: space-between; align-items: center; gap: 20px; flex-wrap: wrap; }
        .total-amount { font-size: 32px; font-weight: 800; color: #f1c40f; }
        #database-list {
            margin-top: 15px;
            max-height: 350px;
            overflow-y: auto;
            background: #fff;
            border-radius: 6px;
            display: none;
            border: 1px solid var(--border);
            padding: 10px;
        }
        .db-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            gap: 8px;
            padding: 8px 12px;
            border-bottom: 1px solid #eee;
            font-size: 13px;
        }
        .db-item input {
            width: 90px;
            padding: 5px;
        }
        .btn-sm {
            background: var(--primary);
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
        }
        @media (max-width: 850px) {
            .manager-grid { grid-template-columns: 1fr; }
            .calc-table thead { display: none; }
            .calc-table tr { display: block; border: 1px solid var(--border); margin-bottom: 15px; padding: 10px; border-radius: 8px; }
            .calc-table td { display: flex; justify-content: space-between; align-items: center; border: none; padding: 5px 0; }
            .calc-table td::before { content: attr(data-label); font-weight: bold; font-size: 12px; color: var(--primary); margin-right: 10px; }
        }
    </style>
</head>
<body>

<main class="main-container">
    <header>
        <h1>Калькулятор расходных материалов</h1>
        <p>ИНЖЕНЕРНО-СТРОИТЕЛЬНЫЙ ХОЛДИНГ &laquo;КЛИМАТ-ЭКСПЕРТ&raquo;</p>
    </header>

    <section class="catalog-manager">
        <h3 style="margin:0 0 15px 0; font-size: 16px; color: var(--primary);">📦 База данных (каталог)</h3>
        <div class="manager-grid">
            <div><input id="new-item-name" placeholder="Название товара" type="text" /></div>
            <div><input id="new-item-unit" placeholder="Ед. (м/шт)" type="text" /></div>
            <div><input id="new-item-price" placeholder="Цена" step="0.01" type="number" /></div>
            <button class="btn btn-save" onclick="addNewProduct()">Сохранить</button>
            <button class="btn" onclick="toggleDbList()" style="background: #95a5a6; color: white;">Все товары</button>
        </div>
        <div id="database-list"></div>
    </section>

    <button class="btn btn-add-row" onclick="addRow()">+ Добавить позицию</button>

    <div class="table-wrapper">
        <table class="calc-table">
            <thead>
                <tr>
                    <th style="width: 45%;">Товар</th>
                    <th style="width: 10%; text-align: center;">Ед.</th>
                    <th style="width: 15%;">Кол-во</th>
                    <th style="width: 15%;">Цена</th>
                    <th style="width: 15%; text-align: right;">Сумма</th>
                    <th style="width: 5%;">&nbsp;</th>
                </tr>
            </thead>
            <tbody id="tableBody"></tbody>
        </table>
    </div>

    <div class="sticky-footer-panel">
        <div class="footer-flex">
            <div><span style="opacity: 0.8;">ИТОГО:</span> <span class="total-amount" id="grandTotal">0.00 ₽</span></div>
            <div style="display: flex; gap: 10px;">
                <button class="btn" onclick="syncPricesFromCatalog()" style="background-color: #ff9800; color: white;">🔄 Обновить цены</button>
                <button class="btn" onclick="clearAll()" style="background-color: #7f8c8d; color: white;">Очистить</button>
                <button class="btn" onclick="copyToClipboard()" style="background-color: var(--success); color: white;">Скопировать</button>
            </div>
        </div>
    </div>
</main>

<script>
    const initialPriceList = [
        { name: "Кабель-канал 'РУВИНИЛ' 74х55х2000мм", unit: "м", price: 232 },
        { name: "Прямой ввод в стену 'РУВИНИЛ' 74х55 мм.", unit: "шт.", price: 146 },
        { name: "Угол внутренний 'РУВИНИЛ' РКК-74х55", unit: "шт.", price: 136 },
        { name: "Угол для кабель-канала 'РУВИНИЛ' УВШ-74х55 наружный", unit: "шт.", price: 153 },
        { name: "Поворот на 90 град. 'РУВИНИЛ' РКК-74х55", unit: "шт.", price: 204 },
        { name: "Поворот гибкий гофрированный для "АРКТИКА РКК-74х55", unit: "шт.", price: 306 },
        { name: "Переходник 'РУВИНИЛ' РКК-74х55 соединительный", unit: "шт.", price: 42 },
        { name: "Ввод в строение РКК-74х55 'РУВИНИЛ'", unit: "шт.", price: 230 },
        { name: "Заглушка для РКК-74х55 'РУВИНИЛ'", unit: "шт.", price: 112 },
        { name: "Тройник накладной 90 град. для РКК 74х55 (белый) РУВИНИЛ", unit: "шт.", price: 281 },
        { name: "Кабель-канал 100х60 L2000 plastic ЭЛЕКОР IEK", unit: "м", price: 263 },
        { name: "Кабель канал 16х16 L2000 plastic ЭЛЕКОР IEK", unit: "м", price: 40 },
        { name: "Медная труба 1/4 ASTM B68 УЗБЕКИСТАН", unit: "м", price: 144 },
        { name: "Медная труба 1/4 ASTM B280 (6,35х0,76)", unit: "м", price: 187 },
        { name: "Медная труба 3/8 ASTM B68 УЗБЕКИСТАН", unit: "м", price: 241 },
        { name: "Медная труба 3/8 ASTM B280 (9,52х0,81)", unit: "м", price: 295 },
        { name: "Медная труба 1/2 ASTM B68 УЗБЕКИСТАН", unit: "м", price: 350 },
        { name: "Медная труба 1/2 ASTM B280 (12,7х0,81)", unit: "м", price: 418 },
        { name: "Медная труба 5/8 ASTM B68 УЗБЕКИСТАН", unit: "м", price: 502 },
        { name: "Медная труба 3/4 ASTM B68 УЗБЕКИСТАН", unit: "м", price: 636 },
        { name: "Медная труба 1/4 (6,35 х 0,61) 15м. Sharq Tubes (UZ)", unit: "м", price: 125 },
        { name: "Труба медная 3/8 (9,52 х 0,65) 15м. Sharq Tubes (UZ)", unit: "м", price: 209 },
        { name: "Труба медная 1/2 (12,70 х 0,70) 15м. Sharq Tubes (UZ)", unit: "м", price: 311 },
        { name: "Гайка 1/4", unit: "шт.", price: 74 },
        { name: "Гайка 3/8", unit: "шт.", price: 113 },
        { name: "Гайка 1/2", unit: "шт.", price: 165 },
        { name: "Гайка 5/8", unit: "шт.", price: 238 },
        { name: "Трубка K-FLEX PE 06x006-2 FRIGO", unit: "м", price: 7.12 },
        { name: "Трубка K-FLEX PE 06x010-2 FRIGO", unit: "м", price: 10 },
        { name: "Трубка K-FLEX PE 06x012-2 FRIGO", unit: "м", price: 11.5 },
        { name: "Трубка K-FLEX PE 06x015-2 FRIGO", unit: "м", price: 13.6 },
        { name: "Трубка K-FLEX PE 06x018-2 FRIGO", unit: "м", price: 15 },
        { name: "Трубка K-FLEX PE 06x022-2 FRIGO", unit: "м", price: 18.5 },
        { name: "Пенофол Black Тип C, толщ. 10, шир. 600, дл. 15", unit: "м", price: 255 },
        { name: "Пенофол Black Тип C, толщ. 5, шир. 600, дл. 30", unit: "м", price: 175 },
        { name: "Пенофол 2000 Тип C, толщ.10, шир.600, дл.15", unit: "м", price: 255 },
        { name: "Пенофол 2000 Тип С, толщ.5, шир.600 дл 30", unit: "м", price: 175 },
        { name: "Провод ПВС 4х1.5 380В Б", unit: "м", price: 83 },
        { name: "Провод ПВС 5х1.5 380В Б", unit: "м", price: 112 },
        { name: "Кабель ВВГ-Пнг(А)-LS 3х2,5ок(N,PE)-0,66", unit: "м", price: 100 },
        { name: "Вилка угловая ВПу11-01-Ст разборная с заземл. 16А", unit: "шт.", price: 83 },
        { name: "Клемма универсальная СМК 222-415 КВТ", unit: "шт.", price: 18 },
        { name: "Клемма универсальная СМК 222-412 КВТ", unit: "шт.", price: 10 },
        { name: "Кронштейн кондиционера 415х450 1,8мм", unit: "шт.", price: 310 },
        { name: "Кронштейн кондиционера L-500х600 1,8мм", unit: "шт.", price: 675 },
        { name: "Кронштейн кондиционера L-700х1000", unit: "шт.", price: 2200 },
        { name: "Кронштейн кондиционера L-1000х1200", unit: "шт.", price: 2780 },
        { name: "Козырек защитный для кондиционера 800*510", unit: "шт.", price: 1952 },
        { name: "Козырек защитный для кондиционера 1000*510", unit: "шт.", price: 3040 },
        { name: "Антивандальная защита кондиционера 800*600*500", unit: "шт.", price: 3402 },
        { name: "Антивандальная защита кондиционера 1000*600*500", unit: "шт.", price: 3402 },
        { name: "Антивандальная защита кондиционера 1000*800*500", unit: "шт.", price: 3965 },
        { name: "Шпилька резьбовая М8*2000мм", unit: "шт.", price: 78 },
        { name: "Траверса монтажная 3000*30*20*0,9", unit: "шт.", price: 267 },
        { name: "Лента перфорированная 20х0,55 прямая (25м)", unit: "шт.", price: 397 },
        { name: "Лента перфорированная CT 20*1,0 (оцинкованная)", unit: "шт.", price: 582 },
        { name: "Лента для защиты термоизоляции 75мм*50м,белая, AVIORA", unit: "шт.", price: 400 },
        { name: "Лента для защиты термоизоляции 75мм*50м,чёрная, AVIORA", unit: "шт.", price: 558 },
        { name: "Лента самоклеящаяся K-flex (3мм*50мм*15м)", unit: "шт.", price: 746 },
        { name: "Скотч алюминиевый ALT 50", unit: "шт.", price: 146 },
        { name: "Скотч алюминиевый ALT 75", unit: "шт.", price: 233 },
        { name: "Скотч армированный серый", unit: "шт.", price: 375 },
        { name: "Изолента ПВХ 19мм*20м белая", unit: "шт.", price: 76 },
        { name: "Дренаж гофрированный D 16 мм (30м)", unit: "шт", price: 785 },
        { name: "Дренаж гофрированный D 20 мм (30м)", unit: "шт", price: 1456 },
        { name: "Дренаж гофрированный D 25 мм (30м)", unit: "шт", price: 1999 },
        { name: "Дренаж металлопласт", unit: "м", price: 62 },
        { name: "Шланг капиллярный 6*9 (бухта 50м)", unit: "м", price: 2 },
        { name: "Регулятор давления конденсации РДК-8.4 (РДКК-33)", unit: "шт.", price: 1248 },
        { name: "Нагреватель дренажа НД-5.5 0,5", unit: "шт.", price: 399 },
        { name: "Нагреватель картера НК-5.4 0,5", unit: "шт.", price: 483 },
        { name: "Припой Castolin S5 (0,5% пруток 2х2х500мм)", unit: "кг", price: 9857 },
        { name: "Дренажная помпа KERNICK VL-15 (22 Л/ЧАС)", unit: "шт.", price: 3094 },
        { name: "Насос дренажный Ballu 10 л/с", unit: "шт.", price: 8884 },
        { name: "Дренажная помпа Caspia Tube", unit: "шт.", price: 3172 },
        { name: "Дренажная помпа Caspia HOME", unit: "шт.", price: 3690 },
        { name: "Помпа Aspen Mini Aqua", unit: "шт.", price: 19240 },
        { name: "Сифон Ballu G-35 Mini", unit: "шт.", price: 968 },
        { name: "Насос дренажный Ballu Machine DС Pump (проточный, 18 л/ч)", unit: "шт.", price: 3498 },
        { name: "Дюбель фасадный KPR-FAST М10*200K с бортом", unit: "шт.", price: 99 },
        { name: "Дюбель фасадный KPR-FAST М10*260K с бортом", unit: "шт.", price: 136 },
        { name: "Дюбель фасадный KPR-FAST М12*260K с бортом", unit: "шт.", price: 170 },
        { name: "Дюбель фасадный KPR-FAST М12*300K с бортом", unit: "шт.", price: 204 },
        { name: "Шуруп для лаг (глухарь) 8*80", unit: "шт.", price: 5 },
        { name: "Дюбель U 12*71 оранж потай", unit: "шт.", price: 2 },
        { name: "Саморез ПШО 4,2*51 FIXER", unit: "шт.", price: 2 },
        { name: "Хладон R 410А в одн. баллоне 11,3 кг FEIYUAN", unit: "шт.", price: 14873 },
        { name: "Хладон R32 в одн. баллоне 9,5 кг FEIYUAN", unit: "шт.", price: 11663 },
        { name: "Хладон R 404А в одн. баллоне 10,9 кг ICELOONG", unit: "шт.", price: 15943 },
        { name: "Хладон R-141b, 10 кг", unit: "шт.", price: 21347 },
        { name: "Насос вакумный (2-ступ. 51 л/мин) VP 215 (Yangy)", unit: "шт.", price: 9443 },
        { name: "Горелка Т-А (Favorcool)", unit: "шт.", price: 1942 },
        { name: "Труборез СТ-274 (1/8'-1 1/8') (DSZH)", unit: "шт.", price: 460 },
        { name: "Труборез WK-428 (3/16'-3/18') (DSZH)", unit: "шт.", price: 460 },
        { name: "Трубогиб пружинный СТ-102-12 3/4 (Favorcool)", unit: "шт.", price: 384 },
        { name: "Трубогиб пружинный СТ-102-10 5/8 (Favorcool)", unit: "шт.", price: 344 },
        { name: "Трубогиб пружинный СТ-102-06 3/8 (Favorcool)", unit: "шт.", price: 275 },
        { name: "Трубогиб пружинный СТ-102-08 1/2 (Favorcool)", unit: "шт.", price: 340 },
        { name: "Трубогиб пружинный CT-102-04 1/4' (Favorcool)", unit: "шт.", price: 123 },
        { name: "PROFcool PB4 1/2+5/8 трубогиб внутренний для медных труб 4м", unit: "шт.", price: 2310 },
        { name: "PROFcool PB4 1/4+3/8 трубогиб внутренний для медных труб 4м", unit: "шт.", price: 1305 },
        { name: "PROFcool TAR-20/90 фиксатор угла поворота из полиамида", unit: "шт.", price: 60 },
        { name: "МАПП газ в баллоне (0,4538)", unit: "шт.", price: 653 },
    ];

    if (!localStorage.getItem('ce_catalog_v4')) {
        localStorage.setItem('ce_catalog_v4', JSON.stringify(initialPriceList));
    }
    let priceListData = JSON.parse(localStorage.getItem('ce_catalog_v4'));

    function parseNumberSafe(val) {
        if (val === undefined || val === null) return 0;
        let str = String(val).trim().replace(',', '.');
        let num = parseFloat(str);
        return isNaN(num) ? 0 : num;
    }

    function saveCatalogToLocal() {
        localStorage.setItem('ce_catalog_v4', JSON.stringify(priceListData));
    }

    function saveRowsToLocal() {
        const rows = [];
        document.querySelectorAll('#tableBody tr').forEach(tr => {
            const name = tr.querySelector('.name-input').value.trim();
            if (name) {
                rows.push({
                    name: name,
                    unit: document.getElementById(`unit-${tr.dataset.id}`).textContent,
                    qty: document.getElementById(`qty-${tr.dataset.id}`).value,
                    price: document.getElementById(`price-${tr.dataset.id}`).value
                });
            }
        });
        localStorage.setItem('ce_calc_rows_v4', JSON.stringify(rows));
    }

    function calculate() {
        let grand = 0;
        document.querySelectorAll('#tableBody tr').forEach(tr => {
            const id = tr.dataset.id;
            const qty = parseNumberSafe(document.getElementById(`qty-${id}`).value);
            const price = parseNumberSafe(document.getElementById(`price-${id}`).value);
            const total = qty * price;
            document.getElementById(`sum-${id}`).textContent = total.toLocaleString('ru-RU', {minimumFractionDigits: 2}) + ' ₽';
            grand += total;
        });
        document.getElementById('grandTotal').textContent = grand.toLocaleString('ru-RU', {minimumFractionDigits: 2}) + ' ₽';
        saveRowsToLocal();
    }

    function handleSearch(id, query) {
        const listDiv = document.getElementById(`list-${id}`);
        listDiv.innerHTML = '';
        const filtered = query === "" 
            ? priceListData 
            : priceListData.filter(item => item.name.toLowerCase().includes(query.toLowerCase()));
        if (filtered.length > 0) {
            listDiv.style.display = 'block';
            filtered.forEach(item => {
                const div = document.createElement('div');
                div.className = 'suggestion-item';
                div.textContent = item.name;
                div.onclick = () => selectItem(id, item);
                listDiv.appendChild(div);
            });
        } else {
            listDiv.style.display = 'none';
        }
    }

    function selectItem(id, item) {
        const tr = document.querySelector(`tr[data-id="${id}"]`);
        tr.querySelector('.name-input').value = item.name;
        document.getElementById(`unit-${id}`).textContent = item.unit;
        document.getElementById(`price-${id}`).value = item.price;
        document.getElementById(`list-${id}`).style.display = 'none';
        calculate();
    }

    function addRow(saved = null) {
        const id = 'id' + Math.random().toString(36).substr(2, 9);
        const tr = document.createElement('tr');
        tr.id = `row-${id}`;
        tr.dataset.id = id;
        tr.innerHTML = `
            <td data-label="Товар">
                <div class="autocomplete-container">
                    <input type="text" class="name-input" autocomplete="off" value="${saved ? saved.name.replace(/"/g, '&quot;') : ''}" 
                        oninput="handleSearch('${id}', this.value)" 
                        onclick="handleSearch('${id}', this.value)"
                        placeholder="Начните вводить название...">
                    <div id="list-${id}" class="suggestions-list"></div>
                </div>
            </td>
            <td data-label="Ед." class="unit-cell" id="unit-${id}">${saved ? saved.unit : '-'}</td>
            <td data-label="Кол-во"><input type="number" id="qty-${id}" value="${saved ? saved.qty : 1}" oninput="calculate()"></td>
            <td data-label="Цена"><input type="number" id="price-${id}" value="${saved ? saved.price : 0}" oninput="calculate()"></td>
            <td data-label="Сумма" class="sum-cell" id="sum-${id}">0.00 ₽</td>
            <td><button class="btn btn-delete" onclick="deleteRow('${id}')">✕</button></td>
        `;
        document.getElementById('tableBody').appendChild(tr);
        calculate();
    }

    function deleteRow(id) { 
        document.getElementById(`row-${id}`).remove(); 
        calculate(); 
    }

    function clearAll() { 
        if(confirm("Очистить таблицу?")) { 
            document.getElementById('tableBody').innerHTML = ''; 
            calculate(); 
        } 
    }

    function addNewProduct() {
        let name = document.getElementById('new-item-name').value.trim();
        let unit = document.getElementById('new-item-unit').value.trim() || 'шт.';
        let price = parseNumberSafe(document.getElementById('new-item-price').value);
        if(!name || isNaN(price) || price <= 0) return alert("Введите корректное название и цену (больше 0)");
        
        const existingIndex = priceListData.findIndex(item => item.name === name);
        if (existingIndex !== -1) {
            if(confirm(`Товар "${name}" уже есть. Обновить цену на ${price} ₽ и единицу измерения на "${unit}"?`)) {
                priceListData[existingIndex].price = price;
                priceListData[existingIndex].unit = unit;
                saveCatalogToLocal();
                alert("Цена и единица обновлены в каталоге");
            }
        } else {
            priceListData.push({name: name, unit: unit, price: price});
            saveCatalogToLocal();
            alert("Новый товар добавлен в базу!");
        }
        document.getElementById('new-item-name').value = '';
        document.getElementById('new-item-unit').value = '';
        document.getElementById('new-item-price').value = '';
    }

    function syncPricesFromCatalog() {
    // 1. Принудительно обновляем базу данных в памяти актуальным массивом из кода
    priceListData = JSON.parse(JSON.stringify(initialPriceList));
    
    // Перезаписываем localStorage, чтобы старые цены там больше не сидели
    // ВНИМАНИЕ: Если у тебя в коде используется ключ v3 или v4 — укажи его здесь вместо v4
    localStorage.setItem('ce_catalog_v4', JSON.stringify(priceListData));

    const rows = document.querySelectorAll('#tableBody tr');
    let updated = 0;

    rows.forEach(tr => {
        const nameInput = tr.querySelector('.name-input');
        // Очищаем имя от лишних пробелов по краям для точного сравнения
        const currentName = nameInput.value.trim().toLowerCase(); 
        if (!currentName) return;

        // Ищем товар в коде, игнорируя регистр букв
        const catalogItem = priceListData.find(item => item.name.trim().toLowerCase() === currentName);
        
        if (catalogItem) {
            const id = tr.dataset.id;
            const priceInput = document.getElementById(`price-${id}`);
            const unitCell = document.getElementById(`unit-${id}`);

            // Принудительно записываем новые данные прямо в ячейки
            priceInput.value = catalogItem.price;
            unitCell.textContent = catalogItem.unit;
            
            // Если менеджер успел выбрать товар, подтягиваем правильное имя из каталога
            nameInput.value = catalogItem.name; 
            updated++;
        }
    });

    // Пересчитываем общую сумму и сохраняем текущие строки
    calculate();
    
    if (updated > 0) {
        alert(`Успешно обновлено позиций: ${updated}. Все цены и единицы измерения взяты напрямую из свежего кода!`);
    } else {
        alert("База данных обновлена. В текущей таблице не найдено совпадений по именам товаров для обновления цен.");
    }
}

    function toggleDbList() {
        const div = document.getElementById('database-list');
        if (div.style.display === 'block') {
            div.style.display = 'none';
            return;
        }
        div.style.display = 'block';
        div.innerHTML = `<div style="padding:8px; background:#f3f6fc; margin-bottom:10px;"><strong>✏️ Редактирование цен каталога:</strong> измените цену и нажмите «Сохранить» рядом с товаром</div>`;
        priceListData.forEach((item, idx) => {
            const row = document.createElement('div');
            row.className = 'db-item';
            row.id = `db-row-${idx}`;
            row.innerHTML = `
                <span style="flex:3; font-size:13px;">${escapeHtml(item.name)}</span>
                <span style="width:60px; text-align:center;">${item.unit}</span>
                <input type="number" id="cat-price-${idx}" value="${item.price}" step="0.01" style="width:90px;">
                <button class="btn-sm" onclick="updateSingleCatalogPrice(${idx})">Сохранить</button>
            `;
            div.appendChild(row);
        });
        const syncBtn = document.createElement('button');
        syncBtn.textContent = '🔄 Синхронизировать цены в таблице с каталогом';
        syncBtn.style.marginTop = '15px';
        syncBtn.style.padding = '10px';
        syncBtn.style.width = '100%';
        syncBtn.style.backgroundColor = '#2e7d32';
        syncBtn.style.color = 'white';
        syncBtn.style.border = 'none';
        syncBtn.style.borderRadius = '6px';
        syncBtn.style.cursor = 'pointer';
        syncBtn.onclick = () => { syncPricesFromCatalog(); div.style.display = 'none'; };
        div.appendChild(syncBtn);
    }

    function updateSingleCatalogPrice(idx) {
        const inputElem = document.getElementById(`cat-price-${idx}`);
        let newPrice = parseNumberSafe(inputElem.value);
        if (isNaN(newPrice) || newPrice < 0) {
            alert("Введите корректную цену (число)");
            return;
        }
        if (newPrice !== priceListData[idx].price) {
            priceListData[idx].price = newPrice;
            saveCatalogToLocal();
            alert(`Цена для "${priceListData[idx].name}" обновлена на ${newPrice} ₽`);
            if (confirm("Обновить эту цену во всех добавленных позициях в таблице?")) {
                syncPricesFromCatalog();
            }
        } else {
            alert("Цена не изменилась");
        }
    }

    function escapeHtml(str) {
        if (!str) return '';
        return str.replace(/[&<>]/g, function(m) {
            if (m === '&') return '&amp;';
            if (m === '<') return '&lt;';
            if (m === '>') return '&gt;';
            return m;
        });
    }

    function copyToClipboard() {
        let text = "🏢 Калькулятор материалов «Климат-Эксперт»\n\n";
        let hasData = false;
        document.querySelectorAll('#tableBody tr').forEach(tr => {
            const id = tr.dataset.id;
            const name = tr.querySelector('.name-input').value.trim();
            if(name) {
                hasData = true;
                const qty = document.getElementById(`qty-${id}`).value;
                const unit = document.getElementById(`unit-${id}`).textContent;
                const sum = document.getElementById(`sum-${id}`).textContent;
                text += `${name} — ${qty} ${unit} = ${sum}\n`;
            }
        });
        if(!hasData) return alert("Таблица пуста!");
        text += `\nИТОГО: ${document.getElementById('grandTotal').textContent}`;
        navigator.clipboard.writeText(text);
        alert("Расчёт скопирован в буфер обмена!");
    }

    document.addEventListener('click', (e) => {
        if (!e.target.classList || !e.target.classList.contains('name-input')) {
            document.querySelectorAll('.suggestions-list').forEach(l => l.style.display = 'none');
        }
    });

    window.onload = () => {
        const savedRows = JSON.parse(localStorage.getItem('ce_calc_rows_v4'));
        if (savedRows && savedRows.length > 0) {
            savedRows.forEach(s => addRow(s));
        } else {
            addRow();
        }
    };
</script>

</body>
</html>
