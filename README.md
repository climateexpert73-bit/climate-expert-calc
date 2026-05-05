<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Калькулятор материалов | Климат-Эксперт</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Segoe UI', Roboto, sans-serif;
            background: #f0f2f5;
            padding: 20px;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            border-radius: 16px;
            padding: 30px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
        }
        h1 {
            color: #0d47a1;
            font-size: 28px;
            text-align: center;
            border-bottom: 3px solid #0d47a1;
            padding-bottom: 15px;
            margin-bottom: 25px;
        }
        .catalog {
            background: #e3f2fd;
            padding: 20px;
            border-radius: 12px;
            margin-bottom: 30px;
        }
        .catalog-grid {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
            align-items: flex-end;
            margin-top: 10px;
        }
        .catalog-grid input, .catalog-grid button {
            padding: 10px;
            border-radius: 8px;
            border: 1px solid #ccc;
        }
        .catalog-grid input {
            flex: 1;
            min-width: 150px;
        }
        button {
            background: #0d47a1;
            color: white;
            border: none;
            cursor: pointer;
            font-weight: 600;
            transition: 0.2s;
        }
        button:hover {
            opacity: 0.9;
        }
        .btn-add {
            background: #0d47a1;
            margin-bottom: 20px;
            padding: 12px 20px;
            font-size: 16px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }
        th {
            background: #f8f9fa;
            color: #0d47a1;
            padding: 12px;
            text-align: left;
            border-bottom: 2px solid #ddd;
        }
        td {
            padding: 10px;
            border-bottom: 1px solid #eee;
            vertical-align: middle;
        }
        input {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 6px;
        }
        .autocomplete {
            position: relative;
        }
        .suggestions {
            position: absolute;
            top: 100%;
            left: 0;
            right: 0;
            background: white;
            border: 1px solid #0d47a1;
            max-height: 250px;
            overflow-y: auto;
            display: none;
            z-index: 1000;
        }
        .suggestions div {
            padding: 8px 12px;
            cursor: pointer;
        }
        .suggestions div:hover {
            background: #e3f2fd;
        }
        .unit-cell {
            color: #666;
            font-size: 14px;
            text-align: center;
        }
        .sum-cell {
            color: #d32f2f;
            font-weight: bold;
            text-align: right;
        }
        .delete-btn {
            background: none;
            color: #999;
            font-size: 20px;
            cursor: pointer;
        }
        .delete-btn:hover {
            color: #d32f2f;
        }
        .footer {
            position: sticky;
            bottom: 20px;
            background: #2c3e50;
            color: white;
            padding: 20px;
            border-radius: 12px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 15px;
            margin-top: 30px;
        }
        .total {
            font-size: 28px;
            font-weight: bold;
            color: #f1c40f;
        }
        .footer button {
            background: #2e7d32;
            padding: 10px 20px;
        }
        @media (max-width: 700px) {
            table thead {
                display: none;
            }
            table tr {
                display: block;
                margin-bottom: 15px;
                border: 1px solid #ddd;
                padding: 10px;
            }
            table td {
                display: flex;
                justify-content: space-between;
                align-items: center;
                border: none;
                padding: 5px 0;
            }
            table td::before {
                content: attr(data-label);
                font-weight: bold;
                color: #0d47a1;
                margin-right: 15px;
            }
        }
    </style>
</head>
<body>
<div class="container">
    <h1>Калькулятор расходных материалов<br><small style="font-size: 14px; color: #d32f2f;">ИНЖЕНЕРНО-СТРОИТЕЛЬНЫЙ ХОЛДИНГ «КЛИМАТ-ЭКСПЕРТ»</small></h1>

    <div class="catalog">
        <h3>📦 База данных (каталог)</h3>
        <div class="catalog-grid">
            <input type="text" id="newName" placeholder="Название товара">
            <input type="text" id="newUnit" placeholder="Ед. (м/шт)" value="шт.">
            <input type="number" id="newPrice" placeholder="Цена" step="0.01">
            <button onclick="addProduct()">Сохранить</button>
            <button onclick="showCatalog()">Все товары</button>
        </div>
        <div id="catalogList" style="display: none; margin-top: 15px; background: white; border-radius: 8px; padding: 10px;"></div>
    </div>

    <button class="btn-add" onclick="addRow()">+ Добавить позицию</button>

    <div style="overflow-x: auto;">
        <table>
            <thead>
                <tr><th>Товар</th><th>Ед.</th><th>Кол-во</th><th>Цена</th><th>Сумма</th><th></th></tr>
            </thead>
            <tbody id="tbody"></tbody>
        </table>
    </div>

    <div class="footer">
        <div><span style="opacity:0.8;">ИТОГО:</span> <span class="total" id="total">0.00 ₽</span></div>
        <div style="display: flex; gap: 10px;">
            <button onclick="syncPrices()">🔄 Обновить цены</button>
            <button onclick="clearTable()">Очистить</button>
            <button onclick="copyResult()">Скопировать</button>
        </div>
    </div>
</div>

<script>
    // ------ КАТАЛОГ (все позиции из вашего списка) ------
    const defaultCatalog = [
        {name:"Кабель-канал 'РУВИНИЛ' 74х55х2000мм", unit:"м", price:227},
        {name:"Прямой ввод в стену 'РУВИНИЛ' 74х55 мм.", unit:"шт.", price:135},
        {name:"Угол внутренний 'РУВИНИЛ' РКК-74х55", unit:"шт.", price:127},
        {name:"Угол для кабель-канала 'РУВИНИЛ' УВШ-74х55 наружный", unit:"шт.", price:150},
        {name:"Поворот на 90 град. 'РУВИНИЛ' РКК-74х55", unit:"шт.", price:190},
        {name:"Переходник 'РУВИНИЛ' РКК-74х55 соединительный", unit:"шт.", price:41},
        {name:"Ввод в строение РКК-74х55 'РУВИНИЛ'", unit:"шт.", price:225},
        {name:"Заглушка для РКК-74х55 'РУВИНИЛ'", unit:"шт.", price:110},
        {name:"Кабель-канал 100х60 L2000 пластик ЭЛЕКОР IEK", unit:"м", price:263},
        {name:"Кабель канал 16х16 L2000 пластик ЭЛЕКОР IEK", unit:"м", price:40},
        {name:"Медная труба 1/4 ASTM B68 УЗБЕКИСТАН", unit:"м", price:144},
        {name:"Медная труба 1/4 ASTM B280 (6,35х0,76)", unit:"м", price:187},
        {name:"Медная труба 3/8 ASTM B68 УЗБЕКИСТАН", unit:"м", price:241},
        {name:"Медная труба 3/8 ASTM B280 (9,52х0,81)", unit:"м", price:295},
        {name:"Медная труба 1/2 ASTM B68 УЗБЕКИСТАН", unit:"м", price:350},
        {name:"Медная труба 1/2 ASTM B280 (12,7х0,81)", unit:"м", price:418},
        {name:"Медная труба 5/8 ASTM B68 УЗБЕКИСТАН", unit:"м", price:502},
        {name:"Медная труба 3/4 ASTM B68 УЗБЕКИСТАН", unit:"м", price:636},
        {name:"Медная труба 1/4 (6,35 х 0,61) 15м. Sharq Tubes (UZ)", unit:"м", price:125},
        {name:"Труба медная 3/8 (9,52 х 0,65) 15м. Sharq Tubes (UZ)", unit:"м", price:209},
        {name:"Труба медная 1/2 (12,70 х 0,70) 15м. Sharq Tubes (UZ)", unit:"м", price:311},
        {name:"Гайка 1/4", unit:"шт.", price:74},
        {name:"Гайка 3/8", unit:"шт.", price:113},
        {name:"Гайка 1/2", unit:"шт.", price:165},
        {name:"Гайка 5/8", unit:"шт.", price:238},
        {name:"Трубка K-FLEX PE 06x006-2 FRIGO", unit:"м", price:7.12},
        {name:"Трубка K-FLEX PE 06x010-2 FRIGO", unit:"м", price:10},
        {name:"Трубка K-FLEX PE 06x012-2 FRIGO", unit:"м", price:11.5},
        {name:"Трубка K-FLEX PE 06x015-2 FRIGO", unit:"м", price:13.6},
        {name:"Трубка K-FLEX PE 06x018-2 FRIGO", unit:"м", price:15},
        {name:"Трубка K-FLEX PE 06x022-2 FRIGO", unit:"м", price:18.5},
        {name:"Пенофол Black Тип C, толщ. 10, шир. 600, дл. 15", unit:"м", price:255},
        {name:"Пенофол Black Тип C, толщ. 5, шир. 600, дл. 30", unit:"м", price:175},
        {name:"Пенофол 2000 Тип C, толщ.10, шир.600, дл.15", unit:"м", price:255},
        {name:"Пенофол 2000 Тип С, толщ.5, шир.600 дл 30", unit:"м", price:175},
        {name:"Провод ПВС 4х1.5 380В Б", unit:"м", price:83},
        {name:"Провод ПВС 5х1.5 380В Б", unit:"м", price:112},
        {name:"Кабель ВВГ-Пнг(А)-LS 3х2,5ок(N,PE)-0,66", unit:"м", price:100},
        {name:"Вилка угловая ВПу11-01-Ст разборная с заземл. 16А", unit:"шт.", price:78},
        {name:"Клемма универсальная СМК 222-415 КВТ", unit:"шт.", price:18},
        {name:"Клемма универсальная СМК 222-412 КВТ", unit:"шт.", price:10},
        {name:"Кронштейн кондиционера 415х450 1,8мм", unit:"шт.", price:310},
        {name:"Кронштейн кондиционера L-500х600 1,8мм", unit:"шт.", price:675},
        {name:"Кронштейн кондиционера L-700х1000", unit:"шт.", price:2200},
        {name:"Кронштейн кондиционера L-1000х1200", unit:"шт.", price:2780},
        {name:"Козырек защитный для кондиционера 800*510", unit:"шт.", price:1952},
        {name:"Козырек защитный для кондиционера 1000*510", unit:"шт.", price:3040},
        {name:"Антивандальная защита кондиционера 800*600*500", unit:"шт.", price:3402},
        {name:"Антивандальная защита кондиционера 1000*600*500", unit:"шт.", price:3402},
        {name:"Антивандальная защита кондиционера 1000*800*500", unit:"шт.", price:3965},
        {name:"Шпилька резьбовая М8*2000мм", unit:"шт.", price:78},
        {name:"Траверса монтажная 3000*30*20*0,9", unit:"шт.", price:267},
        {name:"Лента перфорированная 20х0,55 прямая (25м)", unit:"шт.", price:397},
        {name:"Лента перфорированная CT 20*1,0 (оцинкованная)", unit:"шт.", price:582},
        {name:"Лента для защиты изоляции AVIORA PROFFI 75мм*50м", unit:"шт.", price:400},
        {name:"Лента самоклеящаяся K-flex (3мм*50мм*15м)", unit:"шт.", price:746},
        {name:"Скотч алюминиевый ALT 50", unit:"шт.", price:146},
        {name:"Скотч алюминиевый ALT 75", unit:"шт.", price:233},
        {name:"Скотч армированный серый", unit:"шт.", price:172},
        {name:"Изолента ПВХ 19мм*20м белая", unit:"шт.", price:76},
        {name:"Дренаж гофрированный D 16 мм (30м)", unit:"м", price:25},
        {name:"Дренаж гофрированный D 20 мм (30м)", unit:"м", price:48.5},
        {name:"Дренаж металлопласт", unit:"м", price:62},
        {name:"Шланг капиллярный 6*9 (бухта 50м)", unit:"м", price:2},
        {name:"Регулятор давления конденсации РДК-8.4 (РДКК-33)", unit:"шт.", price:1248},
        {name:"Нагреватель дренажа НД-5.5 0,5", unit:"шт.", price:399},
        {name:"Нагреватель картера НК-5.4 0,5", unit:"шт.", price:483},
        {name:"Припой Castolin S5 (0,5% пруток 2х2х500мм)", unit:"кг", price:9857},
        {name:"Дренажная помпа KERNICK VL-15 (22 Л/ЧАС)", unit:"шт.", price:3094},
        {name:"Насос дренажный Ballu 10 л/с", unit:"шт.", price:8884},
        {name:"Дренажная помпа Caspia Tube", unit:"шт.", price:3172},
        {name:"Дренажная помпа Caspia HOME", unit:"шт.", price:3690},
        {name:"Помпа Aspen Mini Aqua", unit:"шт.", price:19240},
        {name:"Сифон Ballu G-35 Mini", unit:"шт.", price:968},
        {name:"Насос дренажный Ballu Machine DС Pump (проточный, 18 л/ч)", unit:"шт.", price:3498},
        {name:"Дюбель фасадный KPR-FAST М10*200K с бортом", unit:"шт.", price:99},
        {name:"Дюбель фасадный KPR-FAST М10*260K с бортом", unit:"шт.", price:136},
        {name:"Дюбель фасадный KPR-FAST М12*260K с бортом", unit:"шт.", price:170},
        {name:"Дюбель фасадный KPR-FAST М12*300K с бортом", unit:"шт.", price:204},
        {name:"Анкер латунный (цанга) М8", unit:"шт.", price:9},
        {name:"Анкер латунный (цанга) М10", unit:"шт.", price:14},
        {name:"Круг отрезной КО 125*1,2*22", unit:"шт.", price:25},
        {name:"Пена профессиональная огнестойкая всесезонная REDSUN 65 PRO (850мл)", unit:"шт.", price:621},
        {name:"Шуруп сантех.GL8х80(глухарь) 700/50 шт.", unit:"шт.", price:5},
        {name:"Дюбель универсальный (8*72) потайной", unit:"шт.", price:5},
        {name:"Дюбель универсальный 'Рыжий' 6х42 мм", unit:"шт.", price:2},
        {name:"Саморез с пресс-шайбой ПШМ-С 76 х 4.2 мм", unit:"шт.", price:2},
        {name:"Хладон R 410А в одн. баллоне 11,3 кг FEIYUAN", unit:"шт.", price:14873},
        {name:"Хладон R32 в одн. баллоне 9,5 кг FEIYUAN", unit:"шт.", price:11663},
        {name:"Хладон R 404А в одн. баллоне 10,9 кг ICELOONG", unit:"шт.", price:15943},
        {name:"Хладон R-141b, 10 кг", unit:"шт.", price:21347},
        {name:"Масло синтетическое AFROST POE 32 (5л) мет.упак. NEW", unit:"шт.", price:6999},
        {name:"Насос вакумный (2-ступ. 51 л/мин) VP 215 (Yangy)", unit:"шт.", price:9443},
        {name:"Горелка Т-А (Favorcool)", unit:"шт.", price:1942},
        {name:"Труборез СТ-274 (1/8'-1 1/8') (DSZH)", unit:"шт.", price:460},
        {name:"Труборез WK-428 (3/16'-3/18') (DSZH)", unit:"шт.", price:460},
        {name:"Трубогиб пружинный СТ-102-12 3/4 (Favorcool)", unit:"шт.", price:384},
        {name:"Трубогиб пружинный СТ-102-10 5/8 (Favorcool)", unit:"шт.", price:358},
        {name:"Трубогиб пружинный СТ-102-06 3/8 (Favorcool)", unit:"шт.", price:292},
        {name:"Трубогиб пружинный СТ-102-08 1/2 (Favorcool)", unit:"шт.", price:317},
        {name:"Трубогиб пружинный CT-102-04 1/4' (Favorcool)", unit:"шт.", price:226},
        {name:"PROFcool PB4 1/2+5/8 трубогиб внутренний для медных труб 4м", unit:"шт.", price:2310},
        {name:"PROFcool PB4 1/4+3/8 трубогиб внутренний для медных труб 4м", unit:"шт.", price:1305},
        {name:"Виброопоры PROFcool VS-2B110", unit:"шт.", price:738},
        {name:"PROFcool TAR-20/90 фиксатор угла поворота из полиамида", unit:"шт.", price:60},
        {name:"МАПП газ в баллоне (0,4538)", unit:"шт.", price:653},
        {name:"Развальцовочник", unit:"шт.", price:1017}
    ];

    let catalog = JSON.parse(localStorage.getItem('catalog')) || defaultCatalog;
    function saveCatalog() { localStorage.setItem('catalog', JSON.stringify(catalog)); }

    // Функции
    function addRow(saved = null) {
        const id = Date.now() + Math.random();
        const tr = document.createElement('tr');
        tr.dataset.id = id;
        tr.innerHTML = `
            <td data-label="Товар">
                <div class="autocomplete" style="position:relative;">
                    <input type="text" class="name-input" autocomplete="off" value="${saved ? escapeHtml(saved.name) : ''}" oninput="showSuggestions(this, '${id}')" onclick="showSuggestions(this, '${id}')">
                    <div class="suggestions" id="suggest-${id}"></div>
                </div>
            </td>
            <td data-label="Ед." class="unit-cell" id="unit-${id}">${saved ? saved.unit : '-'}</td>
            <td data-label="Кол-во"><input type="number" class="qty" id="qty-${id}" value="${saved ? saved.qty : 1}" oninput="recalc()"></td>
            <td data-label="Цена"><input type="number" class="price" id="price-${id}" value="${saved ? saved.price : 0}" oninput="recalc()"></td>
            <td data-label="Сумма" class="sum-cell" id="sum-${id}">0 ₽</td>
            <td><button class="delete-btn" onclick="this.closest('tr').remove(); recalc();">✕</button></td>
        `;
        document.getElementById('tbody').appendChild(tr);
        recalc();
    }

    function showSuggestions(input, id) {
        const val = input.value.toLowerCase();
        const suggDiv = document.getElementById(`suggest-${id}`);
        if (!val) { suggDiv.style.display = 'none'; return; }
        const filtered = catalog.filter(item => item.name.toLowerCase().includes(val));
        if (filtered.length === 0) { suggDiv.style.display = 'none'; return; }
        suggDiv.innerHTML = filtered.map(item => `<div onclick="selectItem('${id}', '${escapeHtml(item.name)}', '${item.unit}', ${item.price})">${escapeHtml(item.name)}</div>`).join('');
        suggDiv.style.display = 'block';
    }

    window.selectItem = function(id, name, unit, price) {
        const tr = document.querySelector(`tr[data-id="${id}"]`);
        tr.querySelector('.name-input').value = name;
        document.getElementById(`unit-${id}`).innerText = unit;
        document.getElementById(`price-${id}`).value = price;
        document.getElementById(`suggest-${id}`).style.display = 'none';
        recalc();
    };

    function recalc() {
        let total = 0;
        document.querySelectorAll('#tbody tr').forEach(tr => {
            const id = tr.dataset.id;
            const qty = parseFloat(document.getElementById(`qty-${id}`).value) || 0;
            const price = parseFloat(document.getElementById(`price-${id}`).value) || 0;
            const sum = qty * price;
            document.getElementById(`sum-${id}`).innerText = sum.toLocaleString('ru-RU', {minimumFractionDigits:2}) + ' ₽';
            total += sum;
        });
        document.getElementById('total').innerText = total.toLocaleString('ru-RU', {minimumFractionDigits:2}) + ' ₽';
        saveRows();
    }

    function saveRows() {
        const rows = [];
        document.querySelectorAll('#tbody tr').forEach(tr => {
            const name = tr.querySelector('.name-input').value;
            if (name) {
                rows.push({
                    name: name,
                    unit: document.getElementById(`unit-${tr.dataset.id}`).innerText,
                    qty: document.getElementById(`qty-${tr.dataset.id}`).value,
                    price: document.getElementById(`price-${tr.dataset.id}`).value
                });
            }
        });
        localStorage.setItem('savedRows', JSON.stringify(rows));
    }

    function loadRows() {
        const saved = JSON.parse(localStorage.getItem('savedRows'));
        if (saved && saved.length) saved.forEach(r => addRow(r));
        else addRow();
    }

    function addProduct() {
        let name = document.getElementById('newName').value.trim();
        let unit = document.getElementById('newUnit').value.trim() || 'шт.';
        let price = parseFloat(document.getElementById('newPrice').value);
        if (!name || isNaN(price)) return alert('Введите название и цену');
        let exist = catalog.find(p => p.name === name);
        if (exist) {
            if (confirm(`Товар "${name}" уже есть. Обновить цену на ${price}?`)) {
                exist.price = price;
                exist.unit = unit;
                saveCatalog();
                alert('Обновлено');
            }
        } else {
            catalog.push({name, unit, price});
            saveCatalog();
            alert('Добавлено');
        }
        document.getElementById('newName').value = '';
        document.getElementById('newUnit').value = '';
        document.getElementById('newPrice').value = '';
    }

    function showCatalog() {
        const div = document.getElementById('catalogList');
        if (div.style.display === 'block') { div.style.display = 'none'; return; }
        div.innerHTML = '<h4>Редактор каталога (цены)</h4>';
        catalog.forEach((item, idx) => {
            const rowDiv = document.createElement('div');
            rowDiv.style.display = 'flex';
            rowDiv.style.gap = '10px';
            rowDiv.style.marginBottom = '8px';
            rowDiv.style.alignItems = 'center';
            rowDiv.innerHTML = `
                <span style="flex:2;">${escapeHtml(item.name)}</span>
                <span style="width:60px;">${item.unit}</span>
                <input type="number" id="cat-price-${idx}" value="${item.price}" step="0.01" style="width:90px;">
                <button onclick="updateCatPrice(${idx})">Сохранить</button>
            `;
            div.appendChild(rowDiv);
        });
        div.style.display = 'block';
    }

    window.updateCatPrice = function(idx) {
        let newPrice = parseFloat(document.getElementById(`cat-price-${idx}`).value);
        if (!isNaN(newPrice) && newPrice !== catalog[idx].price) {
            catalog[idx].price = newPrice;
            saveCatalog();
            alert('Цена обновлена в каталоге');
            if (confirm('Обновить эти цены во всех строках таблицы?')) syncPrices();
        }
    };

    function syncPrices() {
        let updated = 0;
        document.querySelectorAll('#tbody tr').forEach(tr => {
            const name = tr.querySelector('.name-input').value;
            if (!name) return;
            const catalogItem = catalog.find(c => c.name === name);
            if (catalogItem) {
                const id = tr.dataset.id;
                const oldPrice = parseFloat(document.getElementById(`price-${id}`).value);
                if (oldPrice !== catalogItem.price) {
                    document.getElementById(`price-${id}`).value = catalogItem.price;
                    document.getElementById(`unit-${id}`).innerText = catalogItem.unit;
                    updated++;
                }
            }
        });
        if (updated) { recalc(); alert(`Обновлено ${updated} позиций`); }
        else alert('Все цены актуальны');
    }

    function clearTable() {
        if (confirm('Очистить всю таблицу?')) {
            document.getElementById('tbody').innerHTML = '';
            addRow();
            recalc();
        }
    }

    function copyResult() {
        let text = 'Калькулятор Климат-Эксперт\n\n';
        let has = false;
        document.querySelectorAll('#tbody tr').forEach(tr => {
            const name = tr.querySelector('.name-input').value;
            if (name) {
                const id = tr.dataset.id;
                const qty = document.getElementById(`qty-${id}`).value;
                const unit = document.getElementById(`unit-${id}`).innerText;
                const sum = document.getElementById(`sum-${id}`).innerText;
                text += `${name} — ${qty} ${unit} = ${sum}\n`;
                has = true;
            }
        });
        if (!has) return alert('Таблица пуста');
        text += `\nИТОГО: ${document.getElementById('total').innerText}`;
        navigator.clipboard.writeText(text);
        alert('Скопировано');
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

    document.addEventListener('click', (e) => {
        if (!e.target.classList || !e.target.classList.contains('name-input')) {
            document.querySelectorAll('.suggestions').forEach(s => s.style.display = 'none');
        }
    });

    window.onload = () => { loadRows(); };
</script>
</body>
</html>
