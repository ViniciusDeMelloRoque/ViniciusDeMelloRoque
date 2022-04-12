const axios = require("axios");
const api = require("imersao-bot-cripto-api");
const quantity = 0.001;

const credentials = {
    apiKey:"VBPPgvIwXkrFAGNhwDZjGKI1XtxQUJDFkWLUYGwGUoBhhjpceCEHdvNVVFNag7aO",
    apiSecret:"0TQpxDdZb6o2B9aUCa8dSzXdJzNneTkQA7WVsaGevvgqU506ogyeQJKEIH7MGgKA",
    test: true
}

function calcRSI(closes){
    let gains = 0;
    let lossses = 0;

    for(let i = closes.length - 14; i < closes.length; i++){
        const diff = closes[i] - closes[i - 1];
        if (diff >= 0)
            gains += diff;
        else
            lossses -= diff;    
    }


    const strength = gains / lossses;
    return 100 - (100 /(1 + strength));
}

let bought = true;

async function process() {  
    const symbol = "BTCBUSD";
    
    const response = await axios.get(`https://api.binance.com/api/v3/klines?symbol=${symbol}&interval=1m`);

    const closes = response.data.map(candle => parseFloat(candle[4]));
    const rsi = calcRSI(closes);
    console.log(rsi);


    if(rsi > 70 && bought){
        console.log("Sobrecomprado");
        const sellResult = await api.sell(credentials,symbol,quantity);
        console.log(sellResult);
        bought = false;
    }    
    else if(rsi < 30 && !bought){
        console.log("Sobrevendido");
        const buyResult = await api.buy(credentials,symbol,quantity);
        console.log(buyResult);
        bouth = true;
    }    
     
}

setInterval(process, 60000)

process();




