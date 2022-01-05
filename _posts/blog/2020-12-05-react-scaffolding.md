---
layout: post
title: react项目初始化脚手架
category: blog
description: react项目初始化脚手架
---

### 为方便工业项目初始化开发的一个项目初始化规范脚手架 npm 包

> bin/index.js

```javascript
#!/usr/bin/env node
process.env.NODE_PATH = __dirname + "/../node_modules/";
const chalk = require("chalk");
const checkCmd = require("../index");
const program = require("commander");
const parseArgv = require("./parseArgv");
// const cx = require('../dist/lib/index').default;
const version = require("../package.json").version;

// == promise报错直接抛出
process.on("unhandledRejection", (reason) => {
  throw reason;
});

//解析命令行
const { cmd, args, selfArgs } = parseArgv(process.argv);
const checkPassOption = checkCmd({ cmd, args, selfArgs });
if (checkPassOption) {
  program
    .version(version)
    .usage("<command> [options] 快速启动项目")
    .option("-l, --list", "查看所有的模版")
    .option("-i, --init", "初始化模板")
    .option("-es, --eslint", "初始化eslint配置")
    .parse(process.argv);
  // const options = program.opts();
  // cx({ ...options });
} else {
  console.log(chalk.red(`xtf-cli不支持该命令`));
}
```

> src/lib/init.js

```javascript
import ora from 'ora';
import inquirer from 'inquirer';
import chalk from 'chalk';
import request from 'request';
import download from 'download-git-repo';

export default async () => {
    request(
        {
            url: 'https://api.github.com/users/xtf-template-library/repos',
            headers: {
                'User-Agent': 'edu-test-cli',
            },
        },
        (err, res, body) => {
            if (err) {
                console.log(chalk.red('查询模版列表失败'));
                console.log(chalk.red(err));
                process.exit();
            }

            const requestBody = JSON.parse(body);
            if (Array.isArray(requestBody)) {
                let tplNames = [] as any;
                requestBody.forEach((repo) => {
                    tplNames.push(repo.name);
                });

                let promptList = [
                    {
                        type: 'list',
                        message: '请选择模版',
                        name: 'tplName',
                        choices: tplNames,
                    },
                    {
                        type: 'input',
                        message: '请输入项目名字',
                        name: 'projectName',
                        validate(val) {
                            if (val !== '') {
                                return true;
                            }
                            return '项目名称不能为空';
                        },
                    },
                ];
                inquirer.prompt(promptList).then((answers) => {
                    let ind = requestBody.find(function (ele) {
                        return answers.tplName == ele.name;
                    });
                    let gitUrl = `${ind.full_name}#${ind.default_branch}`,
                        defaultUrl = './',
                        projectUrl = `${defaultUrl}/${answers.projectName}`,
                        spinner = ora('\n 开始生成项目，请等待...');
                    spinner.start();
                    download(gitUrl, projectUrl, (error) => {
                        spinner.stop();
                        if (error) {
                            console.log('模版下载失败……');
                            console.log(error);
                            process.exit();
                        }
                        console.log(chalk.green(`\n √ ${answers.projectName} 项目生成完毕!`));
                        console.log(`\n cd ${answers.projectName} && npm install \n`);
                    });
                });
            } else {
                console.error(requestBody.message);
            }
        },
    );
};
```
