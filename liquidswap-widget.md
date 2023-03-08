# LiquidSwap Widget

A web component custom element which can be embedded to any frontend application or even plain html / js / css. Using full strength of [https://www.npmjs.com/package/@pontem/liquidswap-sdk](https://www.npmjs.com/package/@pontem/liquidswap-sdk) widget can provide swap operations with multiple wallets.

## Installation

```bash
yarn add @pontem/liquidswap-widget
```

or

```bash
npm install @pontem/liquidswap-widget
```

## Usage

Function `loadWidget` accepts widgets HTML tag name as string. It can be custom name but should be in kebab case. We recommend to use ‘liquidswap-widget’ name. Passed name should be exactly the same as tag name.

*   React

    ```tsx
    import React, { useLayoutEffect } from 'react';
    import { loadWidget } from '@pontem/liquidswap-widget';

    export const Widget = () => {
      useLayoutEffect(() => {
        loadWidget('liquidswap-widget');
      }, []);
      return (
        <div className="'Your Wrapper className'">
          <liquidswap-widget/>
        </div>
      );
    };
    ```
*   Vue

    ```typescript
    <template>
      <div class="'Your Wrapper class name'">
        <liquidswap-widget>
      </div>
    </template>

    <script setup lang="ts">
      import { loadWidget } from '@pontem/liquidswap-widget';
      loadWidget('liquidswap-widget');
    </script>
    ```
*   Any other framework / lib

    1. Make sure you added html tag `liquidswap-widget` into app.

    ```html
    <liquidswap-widget></liquidswap-widget>
    ```

    1. Import `loadWidget` function from npm and run with passing html tag name in kebab case;

    ```
    import { loadWidget } from '@pontem/liquidswap-widget';loadWidget('liquidswap-widget');
    ```

## When Loaded

You can listen widget and add some logic (if necessary) when widget is embedded:

```jsx
// customElements defined at global window object. 
customElements.whenDefined('liquidswap-widget').then(() => {
     // here custom logic
});
```

## Live Example

Widget example published to github pages:

[https://pontem-network.github.io/liquidswap-widget/](https://pontem-network.github.io/liquidswap-widget/)

## LiquidSwap Widget as Dapp (native wallet application) <a href="#wallet-integration" id="wallet-integration"></a>

LiquidSwap Widget could be used as Dapp inside wallet.

In that case web-custom-element needs several properties:

* Account address of wallet;

```tsx
const dataAccount = '0x15fd61229f6e12b51adbff45b7b74310c7eaf9c24ef8c13b653c8f2a07bc1d14';
```

* Network information: name and chainId;

```tsx
const dataNetwork = { name: 'mainnet', chainId: '1' };
```

* Transaction info: status and hash;

```tsx
interface ITransactionStatus {
  status: 'pending' | 'success' | 'error' | 'rejected';
  hash: string | null
};
const transactionStatus: ITransactionStatus = { status: 'pending', hash: null };
```

* Properties should be passed as strings (data-attributes), so to pass ‘Object’ variable you need to use JSON.stringify() on passing ‘Object’.
* Properties will be extruded with JSON.parse() inside widget;
* Properties are reactive, so any change to props will update widget’s internal store;

Also, widget will dispatch to Custom Events:

1. ‘**signAndSubmitTransaction**’ - this event will return CustomEvent with ‘detail’ property containing ready for wallet transaction payload. Just accept props inside handler and you will find prop ‘detail’ with full transaction payload;
2. ‘**transactionProcessed**’ - this event will be fired after widget accepts hash and status of transaction and correctly resolves it with.

<details>

<summary>Example in React-JSX</summary>

```jsx
import React, { useLayoutEffect, useRef, useState } from 'react';
import { loadWidget } from '@pontem/liquidswap-widget';

export const Widget = () => {
  const [dataNetwork, setDataNetwork] = useState(
    { name: 'mainnet', chainId: '1' }
  );
  const [dataAccount, setDataAccount] = useState(
    '0x019b68599dd727829dfc5036dec02464abeacdf76e5d17ce43352533b1b212b8'
  );
  const [transactionStatus, setTransactionStatus] = useState<{ 
    status: string,
    hash: string | null
  }>({ status: 'pending', hash: null });
  
  const ref = useRef();
  const transactionHandler = (props: CustomEvent) => {
    // props.detail will be contain payload. 
  };
  const processedHandler = (event: CustomEvent) => {
    // this event will be fired when user close modal with transaction status
    // neither it's success or error. After that point we do not need to 
    // provide hash or status. So it should be set to initial State: 
    setTransactionStatus({ status: 'pending', hash: null });
  };
  useLayoutEffect(() => {
    loadWidget('liquidswap-widget');
    // customElements available as window.customElements 
    customElements.whenDefined('liquidswap-widget').then(() => {
      if (ref.current) {
        const nodeElement = ref.current as unknown as Element;
        nodeElement.addEventListener(
	  'signAndSubmitTransaction',
	  transactionHandler
	);
        nodeElement.addEventListener(
	  'transactionProcessed',
          processedHandler
	);
      }
    });
  }, []);
  
  return (
     <liquidswap-widget
        ref={ref}
        data-network={JSON.stringify(dataNetwork)}
	data-account={dataAccount}
	data-transaction={JSON.stringify(transactionStatus)}
     />
   )
};
```

</details>

Note: When this properties are passed to widget - it treats Parent element as ‘wallet environment’, so connect / disconnect wallet flow will be skipped / hidden.
