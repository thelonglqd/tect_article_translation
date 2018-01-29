## Cách sử dụng nconf library trong Node
Nguồn:

`https://thejsf.wordpress.com/2015/02/08/node-js-application-configuration-with-nconf/`

Nội dung tiếng Anh:

> Every application should have some sort of configuration facility. This is true even when the application runs from easy-to-edit source files. Editing source files to try different options is tedious and error-prone. Common approaches to configuration include use of environment variables, command-line options, and configuration files. Many applications allow the user to combine these configuration sources according to a precedence hierarchy of some kind. Typically, options passed on the command line are the highest priority, followed by environment variables, followed by config files, followed by any hard-coded defaults.

> I am a little suspicious of the use of environment variables for application configuration. Any application can set environment variables, so it can be very difficult to determine where a given environment variable came from. It’s also the case that any application can read the environment variables. Applications that use environment variables for configuration are a little like people yelling their private business across a crowded room filled with uninterested bystanders. I can see how they might be useful to share system-wide configuration among instances of the same application, or among related applications. Most applications don’t have this need. Those that do can share some other central configuration facility such as a shared configuration file in a standard location.

> I did come across one place where environment variables are needed for a node application: npm start. This standard npm command does not support the use of command-line arguments. Without support for some kind of configuration control using environment variables, administrators using npm start would be forced to edit or replace the standard configuration file to change configuration. As a compromise, I decided to support specifying a different configuration file using an environment variable, but no other use of environment variables. This seems like it would be adequate to allow an administrator to keep a few different configurations around for various situations. Developers wanting to troubleshoot can fall back on use of node server, which does support command-line arguments.

> As with logging, I’m hoping to avoid creating unnecessary dependencies on the configuration approach in other parts of the code. Business components should not even be aware that there is such a thing as “configuration data”. All they need to know is that whoever instantiates them supplies them with the parameters they need. Clearly, the application needs to know that there is configuration data. That said, it still makes sense to keep the interface to the configuration data generic, so that all the statements retrieving settings from configuration don’t have to be changed if the approach to reading the configuration data should change.

> Based on all of the above, here are the user stories for my configuration requirements:

> As a node application developer, I would like the standard configurations to be kept in a separate file so that I can edit them without affecting executable code.
As a node application administrator, I would like to be able to specify a different configuration file on the command line, so that I can easily switch between various configurations.
As a node application administrator, I would like use of environment variables kept to a minimum, so that my node application is not affected by changes other products make to the environment variables.
As a node application developer, I would like to be able to override individual configuration settings on the command line, so that I can troubleshoot suspected configuration problems more easily.
As a node application architect, I would like application configuration to be accessed using standard JavaScript object syntax, so that we can switch to a different configuration reader more easily.
As a node application administrator, I would like to be able to log the effective application configuration, so that I can troubleshoot suspected configuration problems.
It’s no surprise that there are several available components for JavaScript and Node.js that address these pretty basic requirements. One such component is nconf, part of the Flatiron framework, which also contains the Winston logging component featured in my previous post here. nconf can be installed using the following command line, run from the root of your node project:

`npm install nconf --save`

> I based my initial cut at this on Andrewrk’s answer to this question on Stack Overflow. The precedence I’ve chosen is, in descending order of priority: command-line arguments, configuration file, defaults supplied by the application. The configuration file can be specified on the command line or through the use of the environment variable “configFile”, for instance configFile=debugConfig.json npm start. Individual settings can be set on the command-line using node server --settingName

> A note regarding command-line arguments: nconf implements hierarchical configuration through the use of colon as a path delimiter. To set a nested configuration value on the command line, you can simply include the colon in the option name, like so: node server --http:port 31337. This is a bit wonky, and it certainly introduces a dependency between command-line syntax and nconf. It seems to me that this coupling isn’t such a big deal; I can’t see a reason why people would have loads of command lines saved up that needed updating. The main use of the command line for configuration, in my view, is for a developer or expert administrator to tweak a setting or two while troubleshooting an issue.

> The default behavior of nconf is to silently fail if a config file can’t be found. This seems wrong to me, so I’ve changed things to throw an error. There is still a potential for a race condition if the file is deleted after the check for its existence. If this race condition happens, the application will cheerfully proceed without the configuration information in the newly non-existent file. I’m not going to bother eliminating the race condition, because the primary failure mode of the configFile parameter is that the user accidentally passed a bad filename. I figure, if gremlins are deleting configuration files at exactly the right millisecond, let them have their fun.

> As with logging, I’ve implemented a little utility module that hides the details from the rest of the application. On the analogy of “logger”, I decided to name it “configger”:
````javascript
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
````

> Here’s its use in server.js:
````javascript
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
````


