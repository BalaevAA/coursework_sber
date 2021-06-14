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

  const token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIyZGQwOGE3NjEyODE1YWVlYWFiMmM5Mzk5ZTVlYTI2Y2ZlMzUxY2I2ZDY2Zjc0MmMxMzRkYWMxOWEzZDhlOGRjNTM5YmU5MjcwMDQyNjI5OCIsImF1ZCI6IlZQUyIsImV4cCI6MTYyMzc2OTAxNywiaWF0IjoxNjIzNjgyNjA3LCJpc3MiOiJLRVlNQVNURVIiLCJ0eXBlIjoiQmVhcmVyIiwianRpIjoiMDI1MmJiYjQtMGRhZS00Nzc2LWI5NmUtMGRkNTBkMGM4YjlhIiwic2lkIjoiMmQwYjI4ZWItNjkwZC00YmRjLTg4OTctYjRjZWVlYjg0MjBkIn0.OWIRuxT0v7SWAQGsldKdP2PaJTFRaP-2dysBf0J3i3hzBUHmAZ4N5DbA1K3PvDx5RG6BA2AyXWPpF-dQW3Tc6Gq5TmWFmyHGAG5bNkZQBqVOBjH6TBQFfShLHnEeHVEGb8fhk6P-C0eRVpZY_mo55CgO9KotcZDahaaT22DdrF7Exe8UJVa8h9MhWprdEo6FNSmtjTDzZkyLU0N1lHN5fOKXUA-NBwavZSqfKupx8y4_5UiTTVoMRhYc3G3wdEGDOz8v_wE-d2G9KIQSnz6Vp4tiVPNreGAXom831fx1-QV3XCS-JHhKjj4QOQ_TFVbzfczx1PE_R8SkUcF8z2a3JJ4nSwXCQRvGEtzjib8zx8CYz96UtlMuKxanCmSfsGxO9BnMxiHGWUEyigcygSDh48L28NZHzPFiX-3GaSnzqBIhxagl8_kiuX8zbrxLB_oKkamtmZK4f5oLt9nVc2s0cbiMpTmd2T7SCq_Ltrp8vqTLfEnzDsgvBURZC1qYBLB2Yp99w3VHEorpvzuTUivgahBEQR-wsINPCykUdIjiskiYXD7uvMGMMDsyZge6BEKT3YJ4a2OvSi-mfOp6cOJoV-H6uUZfgAMv08RVsTTcITWBn1WXHWrI_e2XYVgscS8zPMyni5Zyx8ayJHNvajD-j09SDdIG6F074d0EQ-PKomQ';
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
      <input id="text-input" type="text" placeholder="Введите, на что потратили" bind:value={text}>
      <input id="price-input" type="number" placeholder="Сколько потратили" bind:value={price}>
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
    max-height: 300px;
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