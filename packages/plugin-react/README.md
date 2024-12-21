# @vitejs/plugin-react [![npm](https://img.shields.io/npm/v/@vitejs/plugin-react.svg)](https://npmjs.com/package/@vitejs/plugin-react)

The default Vite plugin for React projects.

- enable [Fast Refresh](https://www.npmjs.com/package/react-refresh) in development (requires react >= 16.9)
- use the [automatic JSX runtime](https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)
- use custom Babel plugins/presets
- small installation size

```js
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
```

## Options

### include/exclude

Includes `.js`, `.jsx`, `.ts` & `.tsx` by default. This option can be used to add fast refresh to `.mdx` files:

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import mdx from '@mdx-js/rollup'

export default defineConfig({
  plugins: [
    { enforce: 'pre', ...mdx() },
    react({ include: /\.(mdx|js|jsx|ts|tsx)$/ }),
  ],
})
```

> `node_modules` are never processed by this plugin (but esbuild will)

### jsxImportSource

Control where the JSX factory is imported from. Default to `'react'`

```js
react({ jsxImportSource: '@emotion/react' })
```

### jsxRuntime

By default, the plugin uses the [automatic JSX runtime](https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html). However, if you encounter any issues, you may opt out using the `jsxRuntime` option.

```js
react({ jsxRuntime: 'classic' })
```

### babel

The `babel` option lets you add plugins, presets, and [other configuration](https://babeljs.io/docs/en/options) to the Babel transformation performed on each included file.

```js
react({
  babel: {
    presets: [...],
    // Your plugins run before any built-in transform (eg: Fast Refresh)
    plugins: [...],
    // Use .babelrc files
    babelrc: true,
    // Use babel.config.js files
    configFile: true,
  }
})
```

Note: When not using plugins, only esbuild is used for production builds, resulting in faster builds.

#### Proposed syntax

If you are using ES syntax that are still in proposal status (e.g. class properties), you can selectively enable them with the `babel.parserOpts.plugins` option:

```js
react({
  babel: {
    parserOpts: {
      plugins: ['decorators-legacy'],
    },
  },
})
```

This option does not enable _code transformation_. That is handled by esbuild.

**Note:** TypeScript syntax is handled automatically.

Here's the [complete list of Babel parser plugins](https://babeljs.io/docs/en/babel-parser#ecmascript-proposalshttpsgithubcombabelproposals).

## Middleware mode

In [middleware mode](https://vitejs.dev/config/server-options.html#server-middlewaremode), you should make sure your entry `index.html` file is transform

Otherwise, you'll probably get this error:

```import { useState } from "react";
import { ethers } from "ethers";
import "./App.css";

function App() {
  const [num1, setNum1] = useState("");
  const [num2, setNum2] = useState("");
  const [result, setResult] = useState("");
  const [operation, setOperation] = useState("");

  const handleCalculate = async () => {
    if (!num1 || !num2 || !operation) return;

    // Connect to the Ethereum network
    if (typeof window.ethereum !== "undefined") {
      await window.ethereum.enable();
    }
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    const signer = provider.getSigner();

    // Instantiate the contract
    const contract = new ethers.Contract(
      "CONTRACT_ADDRESS_HERE",
      CalculatorABI.abi,
      signer
    );

    // Perform the calculation
    let calculatedResult;
    if (operation === "+") {
      calculatedResult = await contract.add(num1, num2);
    } else if (operation === "-") {
      calculatedResult = await contract.subtract(num1, num2);
    } else if (operation === "*") {
      calculatedResult = await contract.multiply(num1, num2);
    } else if (operation === "/") {
      calculatedResult = await contract.divide(num1, num2);
    }

    setResult(calculatedResult.toString());
  };

  return (
    <div className="container">
      <div className="calculator">
        <h2>Calculator</h2>
        <div className="input-group">
          <input
            type="number"
            value={num1}
            onChange={(e) => setNum1(e.target.value)}
          />
          <select
            value={operation}
            onChange={(e) => setOperation(e.target.value)}
          >
            <option value="+">+</option>
            <option value="-">-</option>
            <option value="*">*</option>
            <option value="/">/</option>
          </select>
          <input
            type="number"
            value={num2}
            onChange={(e) => setNum2(e.target.value)}
          />
        </div>
        <button onClick={handleCalculate}>=</button>
        <div className="result">Result: {result}</div>
      </div>
    </div>
  );
}

export default App;
Uncaught Error: @vitejs/plugin-react can't detect preamble. Something is wrong.
```

## Consistent components exports

For React refresh to work correctly, your file should only export React components. You can find a good explanation in the [Gatsby docs](https://www.gatsbyjs.com/docs/reference/local-development/fast-refresh/#how-it-works).

If an incompatible change in exports is found, the module will be invalidated and HMR will propagate. To make it easier to export simple constants alongside your component, the module is only invalidated when their value changes.

You can catch mistakes and get more detailed warning with this [eslint rule](https://github.com/ArnaudBarre/eslint-plugin-react-refresh).
