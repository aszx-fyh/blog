## 代码分割

```js
const path = require("path");
module.exports = {
	mode: "development",
	entry: {
		app: "./src/app.js",
		print: "./src/print.js",
	},
	output: {
		filename: "[name].[chunkhash].js",
		path: path.resolve(__dirname, "dist"),
		// chunkFilename:"chunk-[name].[id].[chunkhash].js",
	},

	optimization: {
		splitChunks: {
			chunks: "async",//async | all | initial
			minSize: 30000,//模块大小
			minChunks: 1,//
			maxAsyncRequests: 5,
			maxInitialRequests: 3,
			automaticNameDelimiter: "~",
			name: true,
			cacheGroups: {
				commons: {
					name: "commons",
					chunks: "all",
					minChunks: 2,
				},
				vendors: {
					test: /[\\/]node_modules[\\/]/,
					priority: -10,
				},
				default: {
					minChunks: 2,
					priority: -20,
					reuseExistingChunk: true,
				},
			},
		},
	},
};
```

> splitChunks 有默认配置,参考[SplitChunksPlugin](https://www.webpackjs.com/plugins/split-chunks-plugin/)
> 缓存组可以继承和/或覆盖splitChunks.*;中的任何选项。但是test，priority并且reuseExistingChunk只能在高速缓存组级别配置。要禁用任何默认缓存组，请将它们设置为false。
> initial针对初次加载的模块，async针对异步加载，all针对所有的
> 命中多个缓存组,使用priority高的那个