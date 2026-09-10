# WebSocket Agent Configuration

```typescript
import { HttpsProxyAgent } from 'https-proxy-agent';
import { Stocks, StocksWebsocketStreams, STOCKS_WS_STREAMS_PROD_URL } from '@binance/stocks';

const configurationWebsocketStreams = {
    wsURL: STOCKS_WS_STREAMS_PROD_URL,
    agent: new HttpsProxyAgent('your-proxy-url'),
};
const client = new Stocks({ configurationWebsocketStreams });

client.websocketStreams.connect().then(console.log).catch(console.error);
```

---
> Source: [binance/binance-connector-js](https://github.com/binance/binance-connector-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-10 -->
