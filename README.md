# 规范指北

1. 用命令行工具打开你的项目目录
2. 使用以下命令安装必要的包：
    ```bash
    npm install --save-dev @commitlint/cli @commitlint/config-conventional commitizen cz-git husky
    ```
3. 配置 `commitlint` 规则。

    在项目根目录中创建一个名为 `.commitlintrc.js` 的文件，并添加以下内容：

   ```js
   // .commitlintrc.js
   const {execSync} = require("child_process");
   const util = require("util");
   
   // 获取用户名和当前日期
   const author = execSync('git config user.name').toString().replace(/(\r\n\t|\n|\r\t)/g, '').trim()
   const date = new Date().toLocaleDateString()
   
   /** @type {import('cz-git').UserConfig} */
   
   const issuePrefixes = [
     // 如果使用 tapd 作为开发管理
     {value: 'bug', name: 'bug:     标记缺陷已完成'},
     {value: 'story', name: 'story:   标记需求已完成'}
   ];
   
   module.exports = {
     extends: ['@commitlint/config-conventional'],
     rules: {
       'subject-min-length': [2, 'always', 2],
       'subject-empty': [2, 'never'],
     },
     prompt: {
       alias: {fd: 'docs: fix typos'},
       messages: {
         type: '选择你要提交的类型 :',
         scope: '选择一个提交范围（可选）:',
         customScope: '请输入自定义的提交范围 :',
         subject: '填写简短精炼的变更描述 :\n',
         body: '填写更加详细的变更描述（可选）。使用 "|" 换行 :\n',
         breaking: '列举非兼容性重大的变更（可选）。使用 "|" 换行 :\n',
         footerPrefixesSelect: '选择关联tapd内容（可选）:',
         customFooterPrefix: '输入自定义tapd前缀 :',
         footer: '输入对应id (可选) :\n',
         confirmCommit: '是否提交或修改commit ?',
         generatingByAI: '正在生成你的 AI 提交信息...',
         generatedSelectByAI: '选择你认为合适的:',
       },
       types: [
         {value: 'feat', name: 'feat:     新增功能 | A new feature'},
         {value: 'fix', name: 'fix:      修复缺陷 | A bug fix'},
         {value: 'docs', name: 'docs:     文档更新 | Documentation only changes'},
         {value: 'style', name: 'style:    代码格式 | Changes that do not affect the meaning of the code'},
         {value: 'refactor', name: 'refactor: 代码重构 | A code change that neither fixes a bug nor adds a feature'},
         {value: 'perf', name: 'perf:     性能提升 | A code change that improves performance'},
         {value: 'test', name: 'test:     测试相关 | Adding missing tests or correcting existing tests'},
         {value: 'build', name: 'build:    构建相关 | Changes that affect the build system or external dependencies'},
         {value: 'ci', name: 'ci:       持续集成 | Changes to our CI configuration files and scripts'},
         {value: 'revert', name: 'revert:   回退代码 | Revert to a commit'},
         {value: 'chore', name: 'chore:    其他修改 | Other changes that do not modify src or test files'},
       ],
       useEmoji: false,
       emojiAlign: 'center',
       themeColorCode: '38;5;043',
       scopes: ['framework', 'lobby', 'music', 'health', 'work'],
       allowCustomScopes: true,
       allowEmptyScopes: true,
       customScopesAlign: 'bottom',
       customScopesAlias: 'custom',
       emptyScopesAlias: 'empty',
       upperCaseSubject: false,
       markBreakingChangeMode: false,
       allowBreakingChanges: ['feat', 'fix'],
       breaklineNumber: 100,
       breaklineChar: '|',
       skipQuestions: [],
       issuePrefixes: issuePrefixes,
       customIssuePrefixAlign: 'bottom',
       emptyIssuePrefixAlias: 'skip',
       customIssuePrefixAlias: 'custom',
       allowCustomIssuePrefix: false,
       allowEmptyIssuePrefix: true,
       confirmColorize: true,
       maxHeaderLength: Infinity,
       maxSubjectLength: Infinity,
       minSubjectLength: 0,
       scopeOverrides: undefined,
       defaultBody: '',
       defaultIssues: '',
       defaultScope: '',
       defaultSubject: '',
       aiNumber: 5,
       aiQuestionCB: ({
                        maxSubjectLength,
                        diff
                      }) => `用完整句子为以下 Git diff 代码写一个有见解并简洁的 Git 中文提交消息，不加任何前缀，并且内容不能超过 ${maxSubjectLength} 个字符: \`\`\`diff\n${diff}\n\`\`\``,
       formatMessageCB: (messageMod => {
   
         // 生成类型和范围
         let scope = (messageMod.scope === undefined || messageMod.scope.length === 0) ? '' : util.format('(%s)', messageMod.scope)
   
         let typeScope = util.format('%s%s:', messageMod.type, scope)
   
         // 生成header
         let subjects = []
         subjects.push(author, date, messageMod.subject)
         subjects = subjects.filter(Boolean)
         const subject = subjects.join('  ')
   
         // 生成header
         const header = util.format('%s %s', typeScope, subject)
   
         // 生成 body 和 breaking
         const body = messageMod.body
         const breaking = (messageMod.breaking === undefined || messageMod.breaking.length === 0) ? undefined : '重大更新：' + messageMod.breaking
   
         // 生成 footer
         let footer
         if (messageMod.footer !== undefined && messageMod.footer.length > 0) {
           try {
             const valuesArray = issuePrefixes.map(obj => obj.value);
             const values = valuesArray.join('|');
   
             const issuePattern = new RegExp(`(?:${values})`, 'g');
             const issue = messageMod.footer.match(issuePattern)
   
             // 尝试替换掉 颜色符号
             let idString = messageMod.footer.replaceAll(/\x1B\[\w*m/g, '')
             // 替换回车
             idString = idString.replaceAll('\n', '')
   
             const idPattern = /\b\d+\b/g;
             const ids = idString.match(idPattern);
             footer = util.format('--%s=%s --user=%s', issue[0], ids.join(','), author)
           } catch (ex) {
             console.log('生成 foot 失败：' + ex)
   
             footer = messageMod.footer.replaceAll('\n', '')
           }
         }
   
         let result = []
         result.push(header, body, breaking, footer)
         result = result.filter(Boolean)
         const formatterResult = result.join('\n\n')
         return formatterResult
       })
     }
   }
   ```

4. 运行以下命令初始化 `commitizen` 规范。
   
   ```bash
   npx commitizen init cz-git --save-dev --save-exact
   ```
   
5. 配置 `cz-git` 以与 `commitizen` 一起使用。

   在项目根目录中创建一个 `.czrc` 文件，并添加以下内容：

   ```json
   {
      "path": "cz-git"
   }
   ```

6. 配置 `husky` 钩子。

   `husk install`

   在项目根目录中打开 `package.json` 文件，并添加以下内容：

   ```json
   {
      "husky": {
         "hooks": {
            "prepare-commit-msg": "exec < /dev/tty && node_modules/.bin/cz --hook || true",
            "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
         }
      }
   }
   ```

7. 提交代码。

   现在，开发者可以使用 `git commit` 命令来提交代码。

   每次提交时，`husky` 将运行 `prepare-commit-msg` 钩子，并提示开发者填写规范化的提交信息。

   提交完成后，`husky` 将运行 `commit-msg` 钩子，并验证提交消息是否符合 `commitlint` 规则。
