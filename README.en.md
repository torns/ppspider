[中文文档](https://github.com/xiyuan-fengyu/ppspider/blob/master/README.md)

<!-- toc -->

- [Quick Start](#quick-start)
  * [Install NodeJs](#install-nodejs)
  * [Install TypeScript](#install-typescript)
  * [Prepare the development environment](#prepare-the-development-environment)
  * [Download And Run ppspider_example](#download-and-run-ppspider_example)
    + [Clone ppspider_example with IDEA](#clone-ppspider_example-with-idea)
    + [Install npm dependencies](#install-npm-dependencies)
    + [Run tsc](#run-tsc)
    + [Startup ppspider App](#startup-ppspider-app)
- [ppspider System Introduction](#ppspider-system-introduction)
  * [Decorator](#decorator)
    + [@Launcher](#launcher)
    + [@OnStart](#onstart)
    + [@OnTime](#ontime)
    + [@AddToQueue @FromQueue](#addtoqueue-fromqueue)
    + [@JobOverride](#joboverride)
    + [@Serialize Serializable @Transient](#serialize-serializable-transient)
    + [@RequestMapping](#requestmapping)
    + [@Bean @Autowired](#bean-autowired)
    + [@DataUi @DataUiRequest](#dataui-datauirequest)
  * [PuppeteerUtil](#puppeteerutil)
    + [PuppeteerUtil.defaultViewPort](#puppeteerutildefaultviewport)
    + [PuppeteerUtil.addJquery](#puppeteerutiladdjquery)
    + [PuppeteerUtil.jsonp](#puppeteerutiljsonp)
    + [PuppeteerUtil.setImgLoad](#puppeteerutilsetimgload)
    + [PuppeteerUtil.onResponse](#puppeteerutilonresponse)
    + [PuppeteerUtil.onceResponse](#puppeteerutilonceresponse)
    + [PuppeteerUtil.downloadImg](#puppeteerutildownloadimg)
    + [PuppeteerUtil.links](#puppeteerutillinks)
    + [PuppeteerUtil.count](#puppeteerutilcount)
    + [PuppeteerUtil.specifyIdByJquery](#puppeteerutilspecifyidbyjquery)
    + [PuppeteerUtil.scrollToBottom](#puppeteerutilscrolltobottom)
    + [PuppeteerUtil.parseCookies](#puppeteerutilparsecookies)
    + [PuppeteerUtil example](#puppeteerutil-example)
  * [logger](#logger)
- [Debug](#debug)
- [Related Information](#related-information)
  * [jQuery](#jquery)
  * [puppeteer](#puppeteer)
  * [nedb](#nedb)
  * [Angular](#angular)
- [WebUI](#webui)
- [Update Note](#update-note)

<!-- tocstop -->

# Quick Start
## Install NodeJs
http://nodejs.cn/download/  

## Install TypeScript
```
npm install -g typescript
```

## Prepare the development environment
Recommended IDEA(Ultimate version) + NodeJs Plugin  
![Ultimate 版本截图](https://s1.ax1x.com/2018/06/13/CO6qRf.png)  
The NodeJs plugin's version should match with the IDEA's version    
![IDEA 版本](https://s1.ax1x.com/2018/06/13/COcdYt.png)  
![nodejs 插件版本](https://s1.ax1x.com/2018/06/13/COcBSf.png)

[IDEA download](https://www.jetbrains.com/idea/download/)  
[NodeJs Plugin download](http://plugins.jetbrains.com/plugin/6098-nodejs)

## Download And Run ppspider_example
ppspider_example github address  
https://github.com/xiyuan-fengyu/ppspider_example  
### Clone ppspider_example with IDEA   
Warning: git is required and the executable file path of git should be set in IDEA  
![IDEA git 配置](https://s1.ax1x.com/2018/06/13/COcr6S.png)

![IDEA clone from git](https://s1.ax1x.com/2018/06/13/COc6mQ.png)  
![IDEA clone from git](https://s1.ax1x.com/2018/06/13/COccwj.png)

### Install npm dependencies  
Click "Terminal" on the bottom side of IDEA to open a terminal and run the following command 
```
npm install
```

### Run tsc
Run 'tsc' in terminal  
Or  
ContextMenu on package.json -> Show npm Scripts -> Double click 'auto build'   
tsc is a TypeScript compiler which can auto compile the ts file to js file after any ts file change  

### Startup ppspider App
Run lib/quickstart/App.js    
Open http://localhost:9000 in the browser to check the ppspider's status  

# ppspider System Introduction
## Decorator
Declare like    
```
export function TheDecoratorName(args) { ... }
```
Usage  
```
@TheDecoratorName(args)
```
Decorator is similar of look with java Annotation, but Decorator is stronger.  Decorator can provide meta data by parameters and modify the target or descriptor to change behavior of class or method  
In ppspider, many abilities are provided by Decorator    

### @Launcher
```
export function Launcher(theAppInfo: AppInfo) { ... }
```
Launcher of a ppspider app  
Params type  
```
export type AppConfig = { 
    // all cache files and the db file will be saved to the workplace folder. You can save the data files to this folder too.
    workplace: string;  
    
    // file path to save the running status，default is workplace + "/queueCache.json"
    queueCache?: string; 
    
    // import all task class   
    tasks: any[]; 
    
    // import all DataUi class
    dataUis?: any[];   
    
    // all workerFactory instances, only PuppeteerWorkerFactory, NoneWorkerFactory is provided at present
    workerFactorys: WorkerFactory<any>[]; 
    
    // the port for web ui，default 9000  
    webUiPort?: number | 9000;
    
    // logger setting
    logger?: LoggerSetting; 
}
```

### @OnStart
```
export function OnStart(config: OnStartConfig)
```
A job executed once after app start, but you can execute it again by pressing the button in webUI. The button will be found after the queue name in webUI's Queue panel.  

Params type  
```
export type OnStartConfig = {
    // urls to crawl
    urls: string | string[];  
    
    // a workerFactory class whose instance must be declared in @Launcher parameter's property: workerFactorys except NoneWorkerFactory. 
    // PuppeteerWorkerFactory and NoneWorkerFactory are supported at present.
    workerFactory: WorkerFactoryClass;
    
     // if set true, this queue will not run after startup
    running?: boolean;
    
    // config of max paralle num, can be a number or a object with cron key and number value    
    parallel?: ParallelConfig;
    
    // the execute interval between jobs, all paralle jobs share the same exeInterval  
    exeInterval?: number;
    
    // make a random delta to exeInterval
    exeIntervalJitter?: number;
    
    // Task timeout, in milliseconds, default: 300000, negative number means never timeout
    timeout?: number;
    
    // description of this sub task type
    description?: string;
}
```
[@OnStart example](https://github.com/xiyuan-fengyu/ppspider_example/tree/master/src/quickstart)

### @OnTime
```
export function OnTime(config: OnTimeConfig) { ... }
```
A job executed at special times resolved by cron expression  
Params type  
```
export type OnTimeConfig = {
    urls: string | string[];
    cron: string; // cron expression
    workerFactory: Class_WorkerFactory;
    running?: boolean;
    parallel?: ParallelConfig;
    exeInterval?: number;
    exeIntervalJitter?: number;
    timeout?: number;
    description?: string;
}
```
[@OnTime example](https://github.com/xiyuan-fengyu/ppspider_example/tree/master/src/ontime)


### @AddToQueue @FromQueue
Those two should be used together, @AddToQueue will add the function's result to job queue, @FromQueue will fetch jobs from queue to execute     

@AddToQueue
```
export function AddToQueue(queueConfigs: AddToQueueConfig | AddToQueueConfig[]) { ... }
```
@AddToQueue accepts one or multi configs  
Config type：
```
export type AddToQueueConfig = {
    // queue name
    name: string;
    
    // queue provided: DefaultQueue(FIFO), DefaultPriorityQueue
    queueType?: QueueClass;
    
    // filter provided: NoFilter(no check), BloonFilter(check by job's key)
    filterType?: FilterClass;
}
```
You can use @AddToQueue to add jobs to a same queue at multi places, the queue type is fixed at the first place, but you can use different filterType at each place.  

The method Decorated by @AddToQueue shuold return a AddToQueueData like.
```
export type CanCastToJob = string | string[] | Job | Job[];

export type AddToQueueData = Promise<CanCastToJob | {
    [queueName: string]: CanCastToJob
}>
```
If @AddToQueue has multi configs, the return data must like 
```
Promise<{
    [queueName: string]: CanCastToJob
}>
```
PuppeteerUtil.links is a convenient method to get all expected urls, and the return data is just AddToQueueData like.  


@FromQueue
```
export function FromQueue(config: FromQueueConfig) { ... }

export type FromQueueConfig = {
    // queue name
    name: string;
    
    workerFactory: WorkerFactoryClass;

    running?: boolean;

    parallel?: ParallelConfig;
    
    exeInterval?: number;
    
    exeIntervalJitter?: number;
    
    // Task timeout, in milliseconds, default: 300000, negative number means never timeout
    timeout?: number;
    
    description?: string;
}
```
[@AddToQueue @FromQueue example](https://github.com/xiyuan-fengyu/ppspider_example/tree/master/src/queue)

### @JobOverride
```
export function JobOverride(queueName: string) { ... }
```
Modify job info before inserted into the queue.  
You can set a JobOverride just once for a queue.  

A usage scenario is: when some urls with special suffix or parameters navigate to the same page,
you can modify the job key to some special and unique id taken from the url with a JobOverride. After 
that, jobs with duplicate keys will be filtered out.  

Actually, sub task type OnStart/OnTime is also managed by queue whose name just likes OnStart_ClassName_MethodName 
or OnTime_ClassName_MethodName, so you can set a JobOverride to it.   
[JobOverride example](https://github.com/xiyuan-fengyu/ppspider_example/blob/master/src/bilibili/tasks/BilibiliTask.ts)

### @Serialize Serializable @Transient
```
export function Serialize(config?: SerializeConfig) { ... }
export class Serializable { ... }
export function Transient() { ... }
```
@Serialize is used to mark a class, then the class info will keep during serializing and deserializing.
Otherwise, the class info will lose when serializing.   
a class which extends from Serializable supports customized serialize / deserialize， for example: [BitSet](https://github.com/xiyuan-fengyu/ppspider/blob/master/src/common/util/BitSet.ts)    
@Transient is used to mark a field which will be ignored when serializing and deserializing.
Warn: static field will not be serialized.   
These three are mainly used to save running status. You can use @Transient to ignore fields which are not 
related with running status, then the output file will be smaller in size 
and the possible error caused by deep nested object during deserializing will be avoid.  
[example](https://github.com/xiyuan-fengyu/ppspider/blob/master/src/test/SerializeTest.ts)


### @RequestMapping
```
export function RequestMapping(url: string, method: "" | "GET" | "POST" = "") {}
```

@RequestMapping is used to declare the HTTP rest interface, providing the ability to dynamically add tasks remotely. 
Returning the crawl results requires self-implementation (such as asynchronous url callbacks).
[RequestMapping example](https://github.com/xiyuan-fengyu/ppspider_example/blob/master/src/requestMapping/tasks/TestTask.ts)  


### @Bean @Autowired
仿造 java spring @Bean @Autowired 的实现，提供实例依赖注入的功能  
[example](https://github.com/xiyuan-fengyu/ppspider/blob/master/src/test/component/BeanTest.ts)

### @DataUi @DataUiRequest
You can define you own tab page in UI(http://localhost:webPort) by this which can extend support for data visualization and user interaction  
You should import the DataUiClass in @Launcher appConfig.dataUis        

There is a built-in DataUi NedbHelperUi which can support nedb search function        
  
[example 1 DataUi basic usage](https://github.com/xiyuan-fengyu/ppspider/blob/master/src/test/dataUi/test.ts)  
![nedbHelper.png](https://i.loli.net/2019/04/04/5ca5c313d92c4.png)  
![dataUiTest1.png](https://i.loli.net/2019/04/04/5ca5c380b5c04.png)  
![dataUiTest2.png](https://i.loli.net/2019/04/04/5ca5c3d152c72.png)  
  
[example 2 Add Dynamic Job On UI](https://github.com/xiyuan-fengyu/ppspider_example/blob/master/src/dataUi/App.ts)  
![动态任务.png](https://i.loli.net/2019/04/04/5ca5c060b5462.png)  
  
[example 3 Web Page Screenshot](https://github.com/xiyuan-fengyu/ppspider_example/blob/master/src/dataUi/ScreenshotApp.ts)  
Long web page screenshot is also supported     
![网页截图.png](https://i.loli.net/2019/04/04/5ca5c172afff3.png)

## PuppeteerUtil
### PuppeteerUtil.defaultViewPort
set page's view port to 1920 * 1080  

### PuppeteerUtil.addJquery
inject jquery to page, jquery will be invalid after page refresh or navigate, so you should call it after page load.  

### PuppeteerUtil.jsonp
parse json in jsonp string  

### PuppeteerUtil.setImgLoad
enable/disable image load 

### PuppeteerUtil.onResponse
listen response with special url, max listen num is supported

### PuppeteerUtil.onceResponse
listen response with special url just once

### PuppeteerUtil.downloadImg
download image with special css selector

### PuppeteerUtil.links
get all expected urls 

### PuppeteerUtil.count
count doms with special css selector

### PuppeteerUtil.specifyIdByJquery
Find dom nodes by jQuery(selector) and specify random id if not existed，
finally return the id array.    
A usage scenario is:   
In puppeteer, all methods of Page to find doms by css selector finally call 
document.querySelector / document.querySelectorAll which not support some 
special css selectors, howerver jQuery supports. Such as: "#someId a:eq(0)", "#someId a:contains('next')".   
So we can call specifyIdByJquery to specify id to the dom node and keep the special id returned, 
then call Page's method with the special id.  

### PuppeteerUtil.scrollToBottom
scroll to bottom  

### PuppeteerUtil.parseCookies
Parse cookie string to SetCookie Array，than set coookie through page.setCookie(...cookieArr)      
How to get cookie string?   
open interested url in chrome-> press F12 to open devtools -> Application panel -> Storage:Cookies:<SomeUrl> -> cookie detail in the left panel -> choose all by mouse，press Ctrl+c to copy all  
You will get something similar to the following  
```
PHPSESSID	ifmn12345678	sm.ms	/	N/A	35				
cid	sasdasdada	.sm.ms	/	2037-12-31T23:55:55.900Z	27			
```

### PuppeteerUtil example
[PuppeteerUtil example](https://github.com/xiyuan-fengyu/ppspider_example/tree/master/src/puppeteerUtil)


## logger
Use logger.debug, logger.info, logger.warn or logger.error to print log.  
Those functions are defined in src/common/util/logger.ts.  
The output logs contain extra info: timestamp, log level, source file position.  
```
logger.debugValid && logger.debug("test debug");
logger.info("test info");
logger.warn("test warn");
logger.error("test error");
```

# Debug
simple typescript/js code can be debugged in IDEA  

The inject js code can be debugged in Chromium. When building the PuppeteerWorkerFactory instance, set headless = false, devtools = true to open Chromium devtools panel.  
[Inject js debug example](https://github.com/xiyuan-fengyu/ppspider_example/tree/master/src/debug)   
```
import {Launcher, PuppeteerWorkerFactory} from "ppspider";
import {TestTask} from "./tasks/TestTask";

@Launcher({
    workplace: __dirname + "/workplace",
    tasks: [
        TestTask
    ],
    workerFactorys: [
        new PuppeteerWorkerFactory({
            headless: false,
            devtools: true
        })
    ]
})
class App {

}
```

<html>
    <p>
    Add <span style="color: #ff490d; font-weight: bold">debugger;</span> where you want  to debug
    </p>
</html>

```
import {Job, OnStart, PuppeteerWorkerFactory} from "ppspider";
import {Page} from "puppeteer";

export class TestTask {

    @OnStart({
        urls: "http://www.baidu.com",
        workerFactory: PuppeteerWorkerFactory
    })
    async index(page: Page, job: Job) {
        await page.goto(job.url());
        const title = await page.evaluate(() => {
           debugger;
           const title = document.title;
           console.log(title);
           return title;
        });
        console.log(title);
    }

}
```

In addition, when developing DataUi, you can debug it in browser  
![DataUiDebug.png](https://i.loli.net/2019/04/04/5ca5ca3417d49.png)  

# Related Information 
## jQuery
http://www.runoob.com/jquery/jquery-syntax.html  

## puppeteer
https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md  

## nedb
https://github.com/louischatriot/nedb  

## Angular  
https://angular.io/  

# WebUI
open http://localhost:9000 in browser  

Queue panel: view and control app status  
![Queue Help](https://s1.ax1x.com/2018/06/13/COcgTs.png)  

Job panel: search jobs and view details  
![ppspiderJobs.en.png](https://i.loli.net/2018/08/29/5b862f27e2809.png)

# Update Note
2019-04-22 v2.0.1
1. add RequestUtil, Encapsulate request as a promise style call    
2. Modify the declaration of NedbHelperUi.defaultSearchExp to avoid error reporting during ts compilation  

2019-04-04 v2.0.0
1. Multiple task threads of the same queue in the QueueManager no longer share a single wait interval    
2. The QueueManager can forcibly interrupt the execution of a running job (the UI Job panel provides an interactive button)  
    By code  
    ```
    appInfo.eventBus.emit(Events.QueueManager_InterruptJob, JOB_ID, "your interrupt reason");
    ```
3. Status can be saved when ppspider is running    
4. Added @Bean @Autowired decorator to provide dependency injection programming  
5. Added @DataUi @DataUiRequest decorator to provide custom UI tab page, user can customize data visualization page, interactive tool    

2019-01-28 v0.1.22
1. Automatically clear jobs marked as filtered.
2. Add job (status: Fail | Success) to queue by clicking button on the Job UI . 

2018-12-24 v0.1.21
1. Add timeout to task configuration, default 300000ms  
2. Add logs to job during task execution to analysis execution time and failure reasons  

2018-12-10 v0.1.20
1. Fix a bug: duplicate instances of the same user defined Task are created    

2018-11-19 v0.1.19
1. Fix a bug: the logger always print error as '{}'.   
2. You can select multiple items when the condition has a list of options on the UI interface.    
3. Update puppeteer version to 0.10.0.  

2018-09-19 v0.1.18
1. Change the reference address of the exported class in index.ts to the original path, 
    so that the user can locate the source code location in the editor quickly.   
2. Print job detail when an error occurred.  
3. The number of job failures increases after all attempts fail.  
4. The parameter list of several methods declared in the logger to print log is changed to an indefinite parameter list. 
    The format parameter is no longer accepted. The indefinite parameter list is used as the message list, and the messages are automatically divided by '\n';
    The logger automatically converts obj to string using JSON.stringify(obj, null, 4).
5. Add a new property named "exeIntervalJitter" to OnStart, OnTime, FromQueue, it makes a random jitter in a range to the execution interval(exeInterval).
    default: exeIntervalJitter = exeInterval * 0.25  
6. Add a new property named "running" to OnStart, OnTime, FromQueue to control the queue running status, default is true.
    Change the running status like following: 
    ```
    mainMessager.emit(MainMessagerEvent.QueueManager_QueueToggle_queueName_running, queueNameRegex: string, running: boolean)
    ```
7. The nedb related code in JobManager is rewritten with NedbDao  
8. Add a decorator @RequestMapping to declare the HTTP rest interface, providing the ability to dynamically add jobs remotely. 
    Returning the crawl results requires self-implementation (such as asynchronous url callbacks).  

2018-08-24 v0.1.17  
1. In job detail modal, add support to recursively load the parent job detail, 
    add target="_blank" to all 'a' elements to open a new tab when clicked  
2. The push mode of spider system info is changed from periodic to event driven and delayed       
3. Update puppeteer version to 1.7.0    
4. Add NoneWorkerFactory whose init action is not required, it is used when puppeteer is not required  
5. Complement the SSS of logger datetime with 0  
6. The job with empty url is accepted now  

2018-07-31 v0.1.16  
1. Fix bug: no effect after setting maxParallelConfig=0  
2. Update puppeteer version to 1.6.1-next.1533003082302 to solve the bug which causes loss of response in version 1.6.1 of puppeteer  

2018-07-30 v0.1.15  
1. Change the implement of PuppeteerUtil.addJquery  
    Change the inject way of jQuery because Page.addScriptTag is not supported 
    by some website   
2. Fix bug: PuppeteerWorkerFactory.exPage compute a wrong position of js executing error   
3. Fix bug: The CronUtil.next may generate a next execute time which equals to the last job   

2018-07-27 v0.1.14  
1. Add support to logger level check  
    Add support to change logger config  
    Logger config can be set in @launcher  
2. Change the parameter of DefaultJob.datas to optional  
3. Enhance the page created by PuppeteerWorkerFactory, methods such as
    $eval, $$eval, evaluate, evaluateOnNewDocument, evaluateHandle are improved,
    when the inject js executes with error, it will print the detail and position 
    of error
4. Change the implement of serialize    
5. Add comments to source code  
6. Update puppeteer version   

2018-07-24 v0.1.13  
1. Import source-map-support, add logger to print log with source code position

2018-07-23 v0.1.12  
1. Add @Transient Decorator to ignore field when serialization  

2018-07-19 v0.1.11  
1. Set the max wait time to 60 secondes when spider system shutdown,    
    those jobs which are not complete will become fail once and add to queue
    to try again  
2. Fix bug: img url match is incorrect  

2018-07-16 v0.1.8    
1. Implement a serialization util to solve the circular reference question
