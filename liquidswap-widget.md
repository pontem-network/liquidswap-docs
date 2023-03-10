# Liquidswap Widget

The widget is a custom web element that can be embedded in any frontend application, or even in a plain HTML/JS/CSS page. Liquidswap widget leverages the full strength of our [SDK](typescript-sdk.md) and can enable token swaps for any other custom dApp, wallet, etc.

## Installation

```bash
yarn add @pontem/liquidswap-widget
```

or

```bash
npm install @pontem/liquidswap-widget
```

## Usage

The function `loadWidget` accepts a widget HTML tag name as a string. It can be a custom name, but it must be in the kebab case. We recommend to use the name `liquidswap-widget`. The passed name must be exactly the same as the tag name.

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

    1. Make sure to add the html tag `liquidswap-widget` to your app.

    ```html
    <liquidswap-widget></liquidswap-widget>
    ```

    1. Import the `loadWidget` function from npm and run it passing the html tag name in the kebab case;

    ```
    import { loadWidget } from '@pontem/liquidswap-widget';loadWidget('liquidswap-widget');
    ```

## When Loaded

You can listen to the widget and add extra logic (if necessary) once the widget is embedded:

```jsx
// customElements defined as a global window object. 
customElements.whenDefined('liquidswap-widget').then(() => {
     // custom logic goes here
});
```

## Live Example

Here is a widget code example published in Pontem's GitHub directory:

[https://pontem-network.github.io/liquidswap-widget/](https://pontem-network.github.io/liquidswap-widget/)

## Liquidswap Widget as a dApp (native wallet application) <a href="#wallet-integration" id="wallet-integration"></a>

Liquidswap Widget can be used as an in-wallet dApp.

In that case, the web-custom-element needs several properties:

* Wallet account address;

```tsx
const dataAccount = '0x15fd61229f6e12b51adbff45b7b74310c7eaf9c24ef8c13b653c8f2a07bc1d14';
```

* Network info: name and chainId;

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

* Properties should be passed as strings (data-attributes); thus, in order to pass the variable `Object`, you need to use `JSON.stringify()` on the `Object` being passed;
* Properties will be extruded with `JSON.parse()` inside the widget;
* Properties are reactive, so any change to props will update the widget’s internal store;

Also, the widget will dispatch to Custom Events:

1. ‘**signAndSubmitTransaction**’ - this event will return CustomEvent with the ‘detail’ property containing a ready-for-wallet transaction payload. Simply accept the props inside the handler, and you will find prop ‘detail’ with the full transaction payload;
2. ‘**transactionProcessed**’ - this event will be fired after the widget accepts the transaction's hash and status and correctly resolves it with

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
    // props.detail will be the contained payload. 
  };
  const processedHandler = (event: CustomEvent) => {
    // this event will be fired if a user closes the modal with the transaction status
    // with neither success nor error. In this case, we do not need to 
    // provide a hash or status. Therefore, it should be set to the initial State: 
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

Note: When these properties are passed to the widget, it treats the Parent element as ‘wallet environment’, so the connect / disconnect wallet flow will be skipped / hidden.
