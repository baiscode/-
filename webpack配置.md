* webpack.config.js
```javascript
const webpack = require('webpack');
const path = require('path');
// css拆分插件
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
// html插件
const HtmlWebpackPlugin = require('html-webpack-plugin');
// 打包清除插件
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
// 多进程打包插件，减少打包时间
const ThreadLoader = require('thread-loader');
// 分包插件，将第三方插件代码打包成chunk
const TerserWebpackPlugin = require('terser-webpack-plugin');
// 打包时间分析插件
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin');
// css压缩插件
const CssMinimzerPlugin = require('css-minimizer-webpack-plugin');

// thread-loader预热
ThreadLoader.warmup({}, ['babel-loader', 'ts-loader', 'less-loader']);
function resolve(paths) { 
    const pathStr = typeof paths === 'string' ? paths : paths.join('');
    return path.resolve(__dirname, pathStr); 
}
const smp = new SpeedMeasurePlugin();
const config = {
    entry: resolve('../src/index.tsx'),
    output: {
        filename: 'static/js/index.[fullhash].js',
        path: resolve(['..', '/dist']),
    },
    module: {
        rules: [
            {
                test: /\.(js|jsx)$/,
                use: [
                    {
                        loader: 'thread-loader'
                    },
                    {
                        // cacheDirectory生成缓存，减少打包时间
                        loader: 'babel-loader?cacheDirectory',  
                        options: {
                            // 预设
                            presets: [
                                [
                                    '@babel/preset-env',
                                ],
                                // react预设
                                '@babel/preset-react',
                                // typescript预设
                                [
                                    '@babel/preset-typescript',
                                    {
                                        isTSX: true,
                                        allExtensions: true
                                    }
                                ]
                            ],
                            plugins: [
                                // 使用@babel/plugin-transform-runtime兼容老旧浏览器
                                // 使用corejs3时，需要安装runtime-corejs3
                                ['@babel/plugin-transform-runtime', {
                                    corejs: { version: 3 }
                                }]
                            ]
                        }
                    }
                ],
                exclude: /node_modules/,
                include: resolve('..', '/src')
            },
            {
                test: /\.tsx?$/,
                use: [
                    // 缓存loader，需要安装cache-loader
                    {
                        loader: 'cache-loader'
                    },
                    {
                        loader: 'ts-loader'
                    }
                ],
                exclude: /node_modules/,
                include: resolve('..', '/src')
            },
            {
                test: /\.css$/,
                use: [
                    {
                        loader: MiniCssExtractPlugin.loader,
                    },
                    {
                        loader: 'css-loader',
                        options: {
                            // 开启css-module
                            modules: true,
                        }
                    },
                    {
                        loader: 'postcss-loader',
                        options: {
                            postcssOptions: {
                                plugins: [
                                    require('autoprefixer')
                                ]
                            }
                        }
                    }
                ],
                exclude: [/\.module\.css$/, /\.module\.less$/],
                include: resolve('..', '/src')
            },
            {
                test: /\.less$/,
                use: [
                    {
                        loader: MiniCssExtractPlugin.loader,
                    },
                    'css-loader',
                    {
                        loader: 'postcss-loader',
                        options: {
                            postcssOptions: {
                                plugins: [
                                    require('autoprefixer')
                                ]
                            }
                        }
                    },
                    {
                        loader: require.resolve('less-loader')
                    }
                ],
                exclude: [/\.module\.css$/, /\.module\.less$/],
                include: resolve('..', '/src')

            },
            {
                test: /\.(bmp|gif|png|jpe?g)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            // 使用url-loader处理时大小超过5000kb时，使用file-loader处理
                            limit: 5000,
                            fallback: 'file-liader',
                            filename: 'static/assets/images/[name].[fullhash].[ext]'
                        }
                    }
                ]
            },
            {
                test: /\.(eot|woff2?|svg|ttf)([\?]?.*)$/,
                use: [
                    {
                        loader: 'file-loader'
                    }
                ]
            }
        ]
    },
    resolve: {
        extensions: ['.tsx', '.ts', '.js', '.json'],
        alias: {
            '@': resolve(['..', '/src'])
        }
    },
    plugins: [
        new HtmlWebpackPlugin({
            filename: 'index.html',
            template: resolve(['..', '/public/index.html']),
        }),
        new MiniCssExtractPlugin({
            filename: 'static/css/[name].[fullhash].css',
            chunkFilename: 'static/css/[id].[fullhash].css'
        }),
        new CleanWebpackPlugin(),
    ],
    optimization: {
        minimizer: [
            new TerserWebpackPlugin({
                // 开启多进程压缩
                parallel: true,
                extractComments: false,
                terserOptions: {
                    compress: {
                        ecma: 5,
                        drop_console: true,  // 去除console
                        drop_debugger: true,  // 去除debugger
                        reduce_vars: false,  // 不压缩var
                        warnings: false,  // 去除提示
                        inline: 2
                    },
                    output: {
                        ecma: 5,
                        comments: false
                    }
                }
            }),
            new CssMinimzerPlugin()
        ],
        splitChunks: {
            // 第三方代码分包，生成chunks
            minSize: 30000,
            minChunks: 1,
            cacheGroups: {
                vendors: {
                    test: function (module) { 
                        return /react|redux/.test(module.context)
                    },
                    chunks: 'initial',
                    filename: 'static/js/vendors.bundle.js'
                },
                common: {
                    test: function (module) { 
                        return /antd|rc-|moment|lodash|@antd-design/.test(module.context);
                    },
                    chunks: 'initial',
                    filename: 'static/js/common.bundle.js'
                }
            }
        }
    }
}
module.exports = config;
```

* webpack.dev.js
```javascript
const webpack = require('webpack');
const path = require('path');
const { merge } = require('webpack-merge');
const webpackBase = require('./webpack.config');
process.env.NODE_ENV = 'development';

module.exports = merge(webpackBase, {
    mode: 'development',
    devtool: 'cheap-module-source-map',
    devServer: {
        // 热加载
        contentBase: path.resolve('../public'),  // html页面所在的相对目录
        hot: true,
        host: 'localhost',
        port: 8899,
        compress: true,
        historyApiFallback: true,  // 是否可以使用history api
        overlay: true,
    },
    plugins: [
        // 热加载
        new webpack.HotModuleReplacementPlugin()
    ]
})
```

* webpack.prod.js
```javascript
const webpack = require('webpack');
const { merge } = require('webpack-merge');
const webpackBase = require('./webpack.config');
process.env.NODE_ENV = 'production';

module.exports = merge(webpackBase, {
    mode: 'production',
    devtool: 'hidden-source-map',
    optimization: {
        // 开启代码压缩
        minimize: true
    }
})