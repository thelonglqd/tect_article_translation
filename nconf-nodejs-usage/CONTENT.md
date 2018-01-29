## Cách sử dụng nconf library trong Node
Nguồn:

`https://thejsf.wordpress.com/2015/02/08/node-js-application-configuration-with-nconf/`

Nội dung tiếng Anh:

> Every application should have some sort of configuration facility. This is true even when the application runs from easy-to-edit source files. Editing source files to try different options is tedious and error-prone. Common approaches to configuration include use of environment variables, command-line options, and configuration files. Many applications allow the user to combine these configuration sources according to a precedence hierarchy of some kind. Typically, options passed on the command line are the highest priority, followed by environment variables, followed by config files, followed by any hard-coded defaults.

> **_Tất cả các đều nên có cơ chế để config. Điều này đúng ngay cả khi ứng dụng đó thuộc loại dễ sửa source code.
Việc sửa chữa source code để thử một vài options là điều rất dễ dẫn đến lỗi. Cách phổ biến để config gồm có biến môi trường, options của command line và file config. Rất nhiều ứng dụng cho phép sử dụng tất cả các options trên với độ ưu tiên của từng loại khác nhau (ví dụ: biến môi trường sẽ được ưu tiên cao nhất, sau đó đến options của command line, sau đó đến config file, trong trường hợp trùng nhau thì sử dụng giá trị của cái có độ ưu tiên cao nhất). Thông thường thì command line option có độ ưu tiên cao nhất, sau đó đến biến môi trường, sau đó là file config và cuối cùng mới là giá trị được hard-code trong code._**

> I am a little suspicious of the use of environment variables for application configuration. Any application can set environment variables, so it can be very difficult to determine where a given environment variable came from. It’s also the case that any application can read the environment variables. Applications that use environment variables for configuration are a little like people yelling their private business across a crowded room filled with uninterested bystanders. I can see how they might be useful to share system-wide configuration among instances of the same application, or among related applications. Most applications don’t have this need. Those that do can share some other central configuration facility such as a shared configuration file in a standard location.

> **_Tôi là người nghi ngờ việc sử dụng biến môi trường cho việc config ứng dụng. Tất cả mọi ứng dụng đều có thể đặt được những biến môi trường nên thật khó để biết biến môi trường đó từ đâu ra. Việc bất cứ một ứng dụng nào khác có thể đọc được biến môi trường đó cũng là điều đáng lưu ý. Một ứng dụng sử dụng biến môi trường để config giống như một người hô hào ầm ĩ về việc kinh doan cá nhân của mình giữa đám đông những người không có chút quan tâm nào vậy. Tôi biết rằng việc này có thể có ích khi nhiều instances của 1 ứng dụng dùng chung 1 config hay giữa những ứng dụng có liên quan đến nhau, nhưng đa phần ứng dụng không cần tới điều này. Cái nào cần có thể giải quyết bằng cách sử dụng chung 1 config file ở một folder chung nào đó_**

> I did come across one place where environment variables are needed for a node application: `npm start`. This standard npm command does not support the use of command-line arguments. Without support for some kind of configuration control using environment variables, administrators using `npm start` would be forced to edit or replace the standard configuration file to change configuration. As a compromise, I decided to support specifying a different configuration file using an environment variable, but no other use of environment variables. This seems like it would be adequate to allow an administrator to keep a few different configurations around for various situations. Developers wanting to troubleshoot can fall back on use of node server, which does support command-line arguments.

> **_Tôi đã từng config ứng dụng node sử dụng biến môi trường khi chạy lệnh `npm start`. Vì lệnh này không hỗ trợ việc sử dụng commandline options, cũng không support một số tính năng khi sử dụng biến môi trường để config, nên người sử dụng `npm start` bắt buộc phải sửa file để thay đổi config. Để chữa cháy, tôi đã sử dụng nhiều file config khác nhau qua một biến môi trường, nhưng cũng chẳng ai sử dụng biến môi trường nữa cả. Dev muốn quay về sử dụng lệnh `node server` thay vì `npm start`, lệnh này có hỗ trợ việc sử dụng commandline options._**

> As with logging, I’m hoping to avoid creating unnecessary dependencies on the configuration approach in other parts of the code. Business components should not even be aware that there is such a thing as “configuration data”. All they need to know is that whoever instantiates them supplies them with the parameters they need. Clearly, the application needs to know that there is configuration data. That said, it still makes sense to keep the interface to the configuration data generic, so that all the statements retrieving settings from configuration don’t have to be changed if the approach to reading the configuration data should change.

> **_Giống như với việc log (Có lẽ là ở 1 bài viết nào đó của ông này trước đây), tôi tránh việc tạo ra những ràng buộc không cần thiết giữa việc config và source code. Sẽ rất tiện khi configuration data có những interface để lấy ra/thay đổi nhữn biến môi trường dễ dàng (tức là khi những giá trị của config data thay đổi, không cần phải thay đổi code)_**

> Based on all of the above, here are the user stories for my configuration requirements:

> **_Dựa vào tất cả những điều trên, yêu cầu của tôi cho config data bao gồm những thứ sau_**

> As a node application developer, I would like the standard configurations to be kept in a separate file so that I can edit them without affecting executable code.

> **_Là một node dev, tôi muốn config data phải được lưu ở 1 file riêng biệt và tôi có thể thay đổi nó mà không thay đổi source code._**

> As a node application administrator, I would like to be able to specify a different configuration file on the command line, so that I can easily switch between various configurations.

> **_Là một nhà quản trị ứng dụng node đó, tôi muốn tôi có thể khai báo file nhiều loại file config trên command line để tôi có thể dễ dàng chuyển từ config cho môi trường này sang môi trường kia._**

> As a node application administrator, I would like use of environment variables kept to a minimum, so that my node application is not affected by changes other products make to the environment variables.

> **_Là nhà quản trị ứng dụng node đó, tôi muốn sử dụng biến môi trường ít nhất có thể, để ứng dụng node của tôi sẽ không bị ảnh hưởng bởi sự thay đổi biến môi trường từ 1 ứng dụng (trường hợp nhiều ứng dụng chạy trên cùng 1 server vật lý)_**

> As a node application developer, I would like to be able to override individual configuration settings on the command line, so that I can troubleshoot suspected configuration problems more easily.

> **_Là một node dev, tôi muốn có thể override một giá của config bằng commandline để dễ dàng cho việc phát hiện những vấn đề liên quan đến config._**

> As a node application architect, I would like application configuration to be accessed using standard JavaScript object syntax, so that we can switch to a different configuration reader more easily.

> **_Là một kiến trúc sư hệ thống của ứng dụng node này, tôi muốn config cho ứng dụng của mình có thể được truy xuất chỉ bằng cú pháp của object trong Javascript, từ đó, chúng tôi có thể thay đổi config reader một cách dễ dàng._**

> As a node application administrator, I would like to be able to log the effective application configuration, so that I can troubleshoot suspected configuration problems.

> **_Là một quản trị ứng dụng node đó, tôi muốn có thể log ra thông tin về những config đang ảnh hưởng đến ứng dụng, để việc giải quyết những vấn đề mà nghi ngờ liên quan đến config._**

> _It’s no surprise that there are several available components for JavaScript and Node.js that address these pretty basic requirements. One such component is nconf, part of the Flatiron framework, which also contains the Winston logging component featured in my previous post here. nconf can be installed using the following command line, run from the root of your node project: `npm install nconf --save`

> **_Không ngạc nhiên khi NodeJS có khá nhiều những thư viện đáp ứng đủ các yêu cầu cơ bản trên, một trong số đó là `nconf`, thư viện này là 1 phần của framework Flatiron, framework này cũng sử dụng winstion logger được nhắc đến ở bài viết trước của tôi. `nconf` có thể được cài đặt vào project của bạn với câu lệnh sau: `npm install nconf --save`_**

> I based my initial cut at this on Andrewrk’s answer to this question on Stack Overflow. The precedence I’ve chosen is, in descending order of priority: command-line arguments, configuration file, defaults supplied by the application. The configuration file can be specified on the command line or through the use of the environment variable “configFile”, for instance configFile=debugConfig.json npm start. Individual settings can be set on the command-line using node server --settingName

> **_Tôi dựa vào câu trả lời của Andrewk trên stackoverflow để thiết lập độ ưu tiên cho những tuỳ chọn khai báo config. Độ ưu tiên đó là commmand-line arguments > config file > hardcoded config. Config file có thể được khai báo qua command-line arguments hoặc biến môi trường (ví dụ với biến môi trường là `configFile`, câu lệnh sẽ là `configFile=debugConfig.js npm start`). Những config đơn lẻ có thể được khai báo trực tiếp trên command-line arguments._**

> A note regarding command-line arguments: nconf implements hierarchical configuration through the use of colon as a path delimiter. To set a nested configuration value on the command line, you can simply include the colon in the option name, like so: node server --http:port 31337. This is a bit wonky, and it certainly introduces a dependency between command-line syntax and nconf. It seems to me that this coupling isn’t such a big deal; I can’t see a reason why people would have loads of command lines saved up that needed updating. The main use of the command line for configuration, in my view, is for a developer or expert administrator to tweak a setting or two while troubleshooting an issue.

> **_Một lưu ý với việc sử dụng command-line arguments: `nconf` sử dụng config theo thứ bậc, sử dụng dấu hai chấm (:) để truy xuất là những config data bên trong 1 config object JS hoặc nested property của JSON. Ví dụ `node server --http:port 31337`. Cách sử dụng này tạo ra sự phụ thuộc giữa cú pháp command-line và thư viện `nconf`. Nhưng có vẻ nó không phải là vấn đề gì lớn lắm, trường hợp này có thể chỉ đc sử dụng bởi những chuyên gua quản trị ứng dụng để tinh chỉnh 1 số config nhỏ nhằm tìm ra nguyên nhân và giải quyết vấn đề nào đó._**

> The default behavior of nconf is to silently fail if a config file can’t be found. This seems wrong to me, so I’ve changed things to throw an error. There is still a potential for a race condition if the file is deleted after the check for its existence. If this race condition happens, the application will cheerfully proceed without the configuration information in the newly non-existent file. I’m not going to bother eliminating the race condition, because the primary failure mode of the configFile parameter is that the user accidentally passed a bad filename. I figure, if gremlins are deleting configuration files at exactly the right millisecond, let them have their fun.

> **_Default behavior của `nconf` là không báo lỗi gì cả nếu như file config không được tìm thấy, điều này có vẻ không đúng lắm, nên tôi đã sửa nó để ném ra 1 lỗi khi không tìm thấy file. Có khả năng, race condition xảy ra trong trường hợp này, là khi file bị xoá đi ngay sau khi đoạn code kiểm tra sự tồn tại của file trả về không có lỗi. Nếu trường hợp này xảy ra, `nconf` sẽ tiếp tục chạy với confile của 1 file không hề tồn tại ?!. Tôi không bận tâm thêm về việc xử lý race condition trong trường hợp này bởi lí do chính để sinh ra đoạn code này là xử lý việc người dùng `nconf` truyền vào đường dẫn cho file không đúng, còn nếu giả sử như bằng một cách nào đó, file bị delete vài mili giây sau khi đoạn code check không tìm thấy file chạy thì ... cứ kệ mịa nó đi =))._**

> As with logging, I’ve implemented a little utility module that hides the details from the rest of the application. On the analogy of “logger”, I decided to name it “configger”:

> **_Cũng giống như việc cấu hình log, tôi viết một đoạn code riêng để cấu hình cho `nconf` và đặt tên nó là `configger.js`_**

```javascript
/* configger.js
 * Reads configuration using nconf. 
 * Returns a JavaScript object representing the effective configuration.
 */
var configger = require(“nconf“);
var fs = require(“fs“);configger.load = function(defaults) {
    configger.argv().env({whitelist: [“configFile“]});    var configFile = “config.json“;
    if (configger.get(“configFile“)) {
        configFile = configger.get(“configFile“);
    }

    if (!fs.existsSync(configFile)) {
        throw {
            name : “FileNotFoundException“,
            message : “Unable to find configFile “ + configFile
        };
    }

    configger.file(configFile);

    configger.defaults(defaults);

    return configger.get();
}

module.exports = configger;
```

> Here’s its use in server.js:

> **_Đây là cách nó được dùng ở server code_**

```javascript
/*
 * server.js – main entry point for the application
 * The purpose of server.js is to read configuration from configuration sources
 * (defaults, command-line, options files, and so on)
 * and start the webServer.
 */
try {
    var util = require(“util“);
    var logger = require(“./logger“);
    var configger = require(“./configger“);
    var webServer = require(“./webServer“);
    var packageJson = require(“./package.json“);
} catch (e) {
    console.log(“Error initializing application“, e);
    return;
}
var config = configger.load({http:{port:8080}});
logger.addTargets(config.loggingTargets);
logger.info(“app version: “ + packageJson.version);
logger.debug(“config: “ + util.inspect(config, {depth: null}));
logger.debug(“package.json: “ + util.inspect(packageJson,{depth: null}));
webServer.start(config.http.port);
```


