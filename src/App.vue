<template>
  <div>
    <h1>😱😱😱 Генератор описания для авито 😱😱😱</h1>
    <h2>Загрузи файл формата .xlsx</h2>
    <input type="file" @change="handleFileUpload" accept=".xlsx" />
    <button @click="exportToExcel" :disabled="!processedData.length">Сохранить результат в файл</button>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';

export default {
  data() {
    return {
      processedData: [],
    };
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });
        const firstSheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[firstSheetName];
        const jsonData = XLSX.utils.sheet_to_json(worksheet);

        this.generateDescriptions(jsonData);
      };
      reader.readAsArrayBuffer(file);
    },
    priceCheck(desc) {
      return desc.match(/\/ ?1 кг/) ? '<p>Цена указана за 1 кг</p>' : '';
    },
    storageConditions(cat) {
      switch (cat) {
        case 'Рыба охлажденная':
          return '<strong>Условия хранения:</strong> 3 суток при температуре 5С. Не замораживать'
        case 'Рыба свежемороженая':
          return '<strong>Условия хранения:</strong> Хранить при -18°C и ниже, готовить в течение 1-2 дней после разморозки. Повторно не замораживать'
        case 'Рыба копченая':
          return '<strong>Условия хранения:</strong> 30 суток при температуре от 0С до +5С'
        case 'Рыба слабосолёная':
          return '<strong>Условия хранения:</strong> Температура хранения: от -3 до +6 °C. Условия хранения: После вскрытия упаковки продукт хранить при температуре от +2°С до +7°C в течение 48 часов.'
        case 'Водоросли':
        case 'Гребешок':
        case 'Кальмар':
        case 'Креветки':
        case 'Краб':
        case 'Мидии':
        case 'Морской коктейль':
        case 'Осьминог':
          return '<strong>Условия хранения:</strong> 12 месяцев при температуре -18С. Повторно не замораживать.'
        case 'Икра красная':
        case 'Масаго (Мойвы)':
        case 'Тобико (Летучей рыбы)':
        case 'Масло растительное':
          return '<strong>Условия хранения:</strong> 12 месяцев'
        default:
          return ''
      }
    },

    generateDescriptions(data) {
      this.processedData = data.map((item) => ({
        description: `
          <p><strong>${item.title}. ${!this.priceCheck(item.description) ? item.description : ''}</strong></p>
          ${this.priceCheck(item.description)}
          <p>
              <strong>Страна:</strong> ${item.country}<br />
              ${!!item.characteristics
                ? '<strong>Характеристики:</strong>' + item.characteristics + '<br />' 
                : ''}
              ${this.storageConditions(item.category)}
          </p>
          <p><br /></p>
          <p><strong>Скидки!</strong></p>
          <p>😍 Оформите заказ на сайте Главкус и получите дополнительные скидки и предложения (просто введите Главкус в поисковую строку Яндекса и перейдите по первой ссылке):</p>
          <p>
              • Получите скидку в 5% на весь ассортимент товаров при заказе от 15000 руб;<br />
              • Бесплатная доставка при заказе от 10000 руб;<br />
              • Скидка 5% на любые товары в ассортименте по промокоду (промокод высылается вам на почту после покупки любого товара на нашем сайте);
          </p>
          <p>Промокоды вводятся на сайте Главкус в окне оформления заказа.</p>
          <p><br /></p>
          <p><strong>Внимание!</strong> Акции действительны до 1 января, поспешите!</p>
          <p>🤝 Прием заказов с Понедельника по Воскресенье включительно. Если в день отгрузки менеджер не смог до вас дозвониться для подтверждения, заказ может быть отменен.</p>
          <p><br /></p>
          <p><strong>Минимальная сумма заказа 3000 руб.</strong></p>
          <p><br /></p>
          <p><strong>Доставка:</strong></p>
          <p>
              Доставка осуществляется с Понедельника по Субботу включительно, кроме праздничных дней. При размещении заказа до 16:00 доставка осуществляется на следующий день после оформления заказа. Заказы размещенные после 16:00 будут доставлены через день.
          </p>
          <p>
              • Стоимость доставки во все районы Москвы в пределах МКАД - 490 руб<br />
              • За пределами МКАД доставка осуществляется с тарифом 40 руб/км.
          </p>
          <p>Стоимость и возможность доставки в ваш регион необходимо предварительно согласовать с оператором.</p>
          <p><br /></p>
          <p><br /></p>
          <p><strong>🔴 Главкус — главные по вкусу! Интернет-магазин рыбы и морепродуктов.</strong></p>
          <p><strong>🔴 Оформляйте заказ на сайте и получайте выгодные предложения!</strong></p>
        `.trim(),
      }));
    },
    exportToExcel() {
      const worksheet = XLSX.utils.json_to_sheet(this.processedData);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Results');
      XLSX.writeFile(workbook, 'Супер описание.xlsx');
    },
  },
};
</script>

<style>
#app {
  max-width: 1600px !important;
}

body {
  font-family: Arial, sans-serif;
  line-height: 1.6;
}
</style>
