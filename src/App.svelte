<script>
  import { onMount } from 'svelte'
  import {createSmartappDebugger, createAssistant} from '@sberdevices/assistant-client'

  let income = 0;
  let outcome = 0;
  let transactions = [];
  let balance = 0;
  $: balance = income - outcome;

  let text = '';
  let price;
  let ObsceneRegex = /^((у|[нз]а|(хитро|не)?вз?[ыьъ]|с[ьъ]|(и|ра)[зс]ъ?|(о[тб]|под)[ьъ]?|(.\B)+?[оаеи])?-?([её]б(?!о[рй])|и[пб][ае][тц]).?|(н[иеа]|[дп]о|ра[зс]|з?а|с(ме)?|о(т|дно)?|апч)?-?х[уy]([яйиеёю]|ли(?!ган)).?|(в[зы]|(три|два|четыре)жды|(н|сук)а)?-?[б6]л(я(?!(х|ш[кн]|мб)[ауеыио]).?|[еэ][дт]ь?)|(ра[сз]|[зн]а|[со]|вы?|п(р[ои]|од)|и[зс]ъ?|[ао]т)?п[иеё]зд.?|(за)?п[ие]д[аое]?р((ас)?(и(ли)?[нщктл]ь?)?|(о(ч[еи])?)?к|юг)[ауеы]?|манд([ауеы]|ой|[ао]вошь?(е?к[ауе])?|юк(ов|[ауи])?)|муд([аио].?|е?н([ьюия]|ей))|мля([тд]ь)?|лять|([нз]а|по)х|м[ао]л[ао]фь[яию]|(жоп|чмо|гнид)[а-я]|г[ао]ндон|[а-я](с[рс]ать|хрен|хер|дрист|дроч|минет|говн|шлюх|г[а|о]вн)[а-я]|мраз(ь|ота)|сук[а-я])|cock|fuck(er|ing)?$/i;
  function getState() {
    console.log("State was get");
    const state = {
      item_selector: {
        items: [
            income,
            outcome,
            transactions
        ]
      }
    }
    return state;
  }

  const token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIyZGQwOGE3NjEyODE1YWVlYWFiMmM5Mzk5ZTVlYTI2Y2ZlMzUxY2I2ZDY2Zjc0MmMxMzRkYWMxOWEzZDhlOGRjNTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTYyMzk2MDk2NSwiaWF0IjoxNjIzODc0NTU1LCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiZTRjODFmODctZmI0My00ZTdiLTg4ZTUtOGJmYzVjMDE0ZjIwIiwic2lkIjoiOGI4ZTY1MmItMzliYi00NTViLTg2MzYtMWViYTkwZTdiMjU0In0.VUpbmMBF-ZS5msANHlyYt3-ZV0LE2-x0cKYFqzZPWgK8iY5Ezole0wxV5cWRRKf0JEERGoCZ-iuRHfYR7xFg5d1iwHN26iKJTzPmx4Ldz2qMSaRTrtvQCC3I5NzbTVjs9iNIoNBEPy64lTgDZn3gcUBzX54cz_EZwlmzZWc5Yhuoo0vaATVmiEnO6UHpwkppqrcbqFKzgd8cmWe8-Ge_uLVfcn2GihQ3CHLJhDdQoOedAtoUREtiXRDnTPBVxh7RvrxVb-qeapfs69I7db9XuPzPlmaEt8BQCSu33PU9QsfskB6yWrFRBLntXs6ggyI3_3D7za-9hZI9fEWS9y0rPNJGwdp80MjR57FGJpJFRLCxAMaA6VFLOirzIopGcqQzr6VY8D2eRc6PNGMQOF0-vmL759sv4RVahpB6G5_HepTWhme8e6jPfk4xUrW3vFK-8Ub3fUrtZGEaS4A_S8MWwrZoH-4-8cx_rewwUjU8csymO_p65pqUCQHPEaefQk-5cauPxUDjupusls1MhSN5aDlxbE7tHGWkzP4YkbaLg8-NQLu6IxmWe_0jON0X7b6eQV5cZ6vRA5BKzxN7MHFo3Bh5RHOyPq9yP9yC1el3AdIXIwlT0mrHV1lrByG95D31EuV6PA-MbczB06M0jwM2tmp7obnCDnxWvc8QNoPzhJ0';
  const initPhrase = 'Включи Финансовый трекер'

    const init = () => {
      // return createSmartappDebugger({
      //   token,
      //   initPhrase,
      //   getState,
      // })
      return createAssistant({getState});
    }
  const assistant = init();

  assistant.on("data", (event) => {
    console.log('EVENT!!!', event);
    if (event.action.type) {
      if (event.action.type === 'add_transaction') {
        text = event.action.text;
        price = Number(event.action.price);
        addTransaction();
      } else if (event.action.type === 'delete_transaction') {
        if(event.action.number - 1 < transactions.length){
          deleteTransaction(event.action.number-1);
        }
      }
    }
  })

  function addTransaction() {
    if (text.match(ObsceneRegex) !== null) {
      document.getElementById('text-input').style.border='1px solid red';
      return;
    } else {
      document.getElementById('text-input').style.border='';  
    }
    if (text === '') {
      document.getElementById('text-input').style.border='1px solid red';
      return;
    } else {
      document.getElementById('text-input').style.border='';
    }
    if (price === 0 || price === undefined || price === null) {
      document.getElementById('price-input').style.border = '1px solid red';
      return;
    } else {
      document.getElementById('price-input').style.border = '';
    }
    text = text.charAt(0).toUpperCase() + text.slice(1);
    transactions = [...transactions, {
      id: transactions.length,
      text,
      price
    }]
    if (price > 0)
      income += price;
    else
      outcome += -1*price;
    text = '';
    price = undefined;
  }

  function deleteTransaction(index) {
    if (transactions[index].price > 0)
      income -= transactions[index].price;
    else
      outcome += transactions[index].price;
    transactions = [
      ...transactions.slice(0, index),
      ...transactions.slice(index + 1, transactions.length)
    ]

  }
</script>

<main>
	<div id="dashboard">
    <div id="balance">
      <h3>ВАШ БАЛАНС</h3>
      <p>{balance}</p>
    </div>
    <div id="statistics">
      <div>
        <h4>ПОЛУЧИЛИ</h4>
        <p>{income}</p>
      </div>
      <div>
        <h4>ПОТРАТИЛИ</h4>
        <p>{outcome}</p>
      </div>
    </div>
    <div id="history">
      <h3>ИСТОРИЯ</h3>
      <div id="transactions">
        {#each transactions as transaction, ind (transaction.id)}
          <div>
            <div class="cross" on:click={() => deleteTransaction(ind)}>
              &#10006;
            </div>
            <div class="transaction {transaction.price > 0 ? 'positive' : 'negative'}">
              <p>{ind+1}. {transaction.text}</p>
              <p>{transaction.price>0 ? '+' : ''}{transaction.price}</p>
            </div>
          </div>
        {/each}
      </div>
    </div>
    <div id="add-block">
      <h3>ДОБАВИТЬ ТРАНЗАКЦИЮ</h3>
      <input id="text-input" type="text" placeholder="Введите описание транзакции" bind:value={text}>
      <input id="price-input" type="number" placeholder="Введите сумму транзакции" bind:value={price}>
      <button on:click={addTransaction}>Добавить</button>
    </div>
  </div>
</main>

<style>
  main {
    background-color: #f0f0f0;
    width: 100%;
    height: 90%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #dashboard {
    width: 1000px;
    display: grid;
    grid-template-columns: 1fr 300px;
    gap: 20px;
    text-align: center;
    padding: 20px;
  }

  #dashboard > div  {
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
    border-radius: 20px;
    background-color: #FFFFFF;
    padding: 20px;
  }

  #balance h3 {
    margin: 0;
  }

  #balance p {
    margin: 5px 0 0 0;
    font-size: 25px;
  }

  #statistics {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #statistics div {
    padding: 0 20px;
  }

  #statistics div h4 {
    margin: 0 0 5px 0;
  }

  #statistics div p {
    margin: 0;
    font-size: 20px;
  }

  #statistics div:first-of-type {
    border-right: 1px solid gray;
  }

  #transactions {
    max-height: 75%;
    padding: 0 30px;
    overflow: auto;
  }

  #transactions > div {
    display: flex;
    justify-content: start;
    align-items: center;
    margin: 10px 0;
  }

  .cross {
    width: 30px;
    height: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    margin-right: 10px;
    border-radius: 10px;
    border: 1px solid transparent;
  }

  .cross:hover, .cross:focus {
    border: 1px solid red;
  }

  .transaction {
    flex: 1;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
    border-radius: 15px;
    padding: 10px;
  }

  .transaction p {
    margin: 0;
    white-space: normal;
  }

  .transaction p:first-of-type {
    margin-right: 10px;
  }

  .positive {
    border-right: 4px solid green;
  }

  .negative {
    border-right: 4px solid red;
  }

  #add-block {
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  #add-block h3 {
    margin: 0 0 10px 0;
  }

  #add-block input {
    border-radius: 15px;
    padding: 10px;
    margin: 5px 0;
  }

  #add-block input:hover, #add-block input:focus {
    border: 1px solid #000000;
  }

  #add-block button {
    margin-top: 10px;
    cursor: pointer;
  }
  
  @media (max-width: 780px) {
    main {
      align-items: start;
      justify-content: start;
    }
    #dashboard {
      overflow: auto;
      width: 95%;
      max-height: 95%;
      padding: 30px 20px;
      grid-template-columns: repeat(2, 1fr);
      grid-template-rows: 1fr 2fr 2fr;
    }

    #balance {
      grid-column: 1;
      grid-row: 1;
    }

    #statistics {
      grid-column: 2;
      grid-row: 1;
    }

    #history {
      grid-column: 1 / 3;
      grid-row: 2 / 3;
      min-height: 200px;
    }

    #add-block {
      grid-column: 1 / 3;
      grid-row: 3 / 4;
    }
  }

  @media (max-width: 500px) {
    #dashboard {
      grid-template-columns: repeat(2, 1fr);
      grid-template-rows: 1fr 1fr 2fr 2fr;
    }

    #balance {
      grid-column: 1 / 3;
      grid-row: 1;
    }

    #statistics {
      grid-column: 1 / 3;
      grid-row: 2;
    }

    #history {
      grid-column: 1 / 3;
      grid-row: 3 / 4;
    }

    #add-block {
      grid-column: 1 / 3;
      grid-row: 4 / 5;
    }
  }
</style>