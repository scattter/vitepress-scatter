# express中配置log4js
## 1. 安装
毫无疑问, 第一件事就是安装log4js的依赖
```shell
npm install log4js
```

## 2. log4js配置
这里就是配置我们的日志文件, 该文件位于项目根目录, 这里假设文件名为  log-config.js   
里面包含怎么对我们的日志进行分类, 分类后是保存还是在控制台打印出来, 当然里面你也可以自己更改,做自己的配置, 比如存数据库之类啥的
```javascript
import log4js from 'log4js';
log4js.configure({
  replaceConsole: true,
  appenders: {
    // 控制台输出, 只打印, 不保存
    stdout: {
      type: 'stdout'
    },
    // 请求日志, 输出文件为logs/reqLog/
    req: {
      type: 'dateFile',
      // 文件名为= filename + pattern, 设置为alwaysIncludePattern：true
      filename: 'logs/reqLog/',
      pattern: 'req-yyyy-MM-dd.log',
      alwaysIncludePattern: true
    },
    // 错误日志
    err: {
      type: 'dateFile',
      filename: 'logs/errLog/',
      pattern: 'err-yyyy-MM-dd.log',
      alwaysIncludePattern: true
    },
    // 其他日志
    oth: {
      type: 'dateFile',
      filename: 'logs/othLog/',
      pattern: 'oth-yyyy-MM-dd.log',
      alwaysIncludePattern: true
    }
  },
  categories: {
    // appenders: 采用的appender, 取appenders项, level: 设置级别, 想成功调用该类就需要使用logger.{level}去打印
    default: { appenders: ['stdout', 'req'], level: 'debug' },
    err: { appenders: ['stdout', 'err'], level: 'error' },
    oth: { appenders: ['stdout', 'oth'], level: 'info' }
  }
});
const log = {};
// name取categories项
log.getLogger = (name) => log4js.getLogger(name || 'default');
// 用来与express结合
log.useLogger = (app, logger) => {
  // 加载中间件, 在全局默认是default类的log
  app.use(log4js.connectLogger(logger || log4js.getLogger('default'), {
    // 格式化http相关信息, 自定义输出格式
    format: '[:remote-addr :method :url :status :response-timems][:referrer HTTP/:http-version :user-agent]'
  }));
};

// 这里因为我们项目使用的是ES module, 不能使用require, 
// 所以我就定义了一个新的log 暴露出来
export default log;
```

## 3. 全局使用
最后就是在我们的server(或者定义app处使用), 当然你也可以在具体页面使用, 使用和下面差不多, 也是先导入, 然后选择logger就行
```javascript
import express from 'express';
// 导入我们的log配置
import log4js from '../../log-config.js';
// 根据需要获取logger, 全局默认为default类
const logger = log4js.getLogger('default');
// 请求出现错误的话 使用err日志记录
const errLogger = log4js.getLogger('err');

// create an App
const app = express();
// use log4js to record log
log4js.useLogger(app, logger);
// setup pre processing
setup(app);

// createRouter()方法会返回一个数组, 里面是定义的路由,
// 所以这里就先不写了
const routers = createRouter();
routers.forEach((localRouter) => {
  app.get(localRouter.path, (req, res) => {
    // success response
    res.success = (data) => {
      const oResult = {
        code: 0,
        data,
        message: ''
      };
      res.end(JSON.stringify(oResult));
    };
    // error response
    res.error = (code, message) => {
      // 这里404 不会被捕获到
      // 在 Express 中，404 响应不是错误的结果，所以错误处理程序中间件不会将其捕获。
      // 这里捕获请求的错误
      errLogger.error(`path: ${localRouter.path} request: ${req} response: ${code} ${message}`);
      const oResult = {
        code,
        data: null,
        message
      };
      res.end(JSON.stringify(oResult));
    };
    if (!localRouter.module) {
      res.send('Hello World');
    } else {
      localRouter.module(req, res);
    }
  });
});
```

## 4. 效果
最后配置完, 我们可以在项目根目录下看见这三个文件
根据上面的配置,
项目所有的请求信息都会在terminal打印一份, 写入reqLog-xxxx 里面一份;
如果请求出现错误, 那么会在terminal打印一份, 写入errLog-xxxx 里面一份

![nodejs-express-log4](https://cdn.jsdelivr.net/gh/scattter/blogweb/images/nodejs-express-log4.png)