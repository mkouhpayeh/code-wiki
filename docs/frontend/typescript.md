1. Install nmv
https://github.com/coreybutler/nvm-windows/releases
nmv -v

2. Install npm
npm install -g npm

3. Install node.js
https://nodejs.org/en/download/

3. install typescript
npm install -g typescript
vscode project terminal (PowerShell security error)=> Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned

4. Configure TypeScript Compiler
tsc --init

tsconfig.json:
"target": "esnext",

"module": "commonjs"

"rootDir": "./src",

"outDir": "./dist",

"removComments": true,

"noEmitOnError": true, 

"sourceMap" : true => Step by Step Debug

run => tsc 

Debug: Set breakpoint
open debug window > Create a launch.js file > node.js

"program": "${workspaceFolder}/src/index.ts",
"preLaunchTask": "tsc: build - tsconfig.json" : use ts compiler to build config file
