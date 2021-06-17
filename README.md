# Lerna_Practice (21/06/16 작성)

레르나를 한번 사용해보는 repository 참조 https://kdydesign.github.io/2020/08/27/mono-repo-lerna-example/

1. 설치
   `brew install lerna`

2. lerna Repo 구성
   `lerna init --independent`

3. lerna.json
   lerna.json은 lerna를 구성하는 설정 파일이다.

4. package.json
   Root 경로에 있는 package.json의 역할은 공통된 node module들의 devDependencies와 최상위에서 실행될 script들을 명시해준다.

5. 패키지 생성
   `lerna create [package_name]`

6. 공통 종속성
   Root 경로에 모든 패키지가 공통으로 사용될 모듈을 설치하자.
   lerna add라는 명령어를 사용할 수도 있지만 lerna add는 패키지 간 종속성 설치 시 사용하는 걸 추천한다.
   그냥 사용한다 하더라고 Root 경로에 모듈은 설치되겠지만 각 패키지에 dependencies가 걸리게 되어있다.
   그렇기 때문에 Root 경로의 공통 모듈을 npm또는 yarn을 통하여 설치하자.
   `yarn add eslint --dev` X
   `yarn add eslint --dev --ignore-workspace-root-check` O

   NPM 설정 시 그냥 설치하는 듯 `npm install eslint --dev`

7. 개별 종속성
   각 패키지에 모듈을 설치할 때에는 공통 모듈을 설치하는 것과는 다르게 lerna add를 통해서 각 패키지에 설치 할 수 있다. 이 과정에서 hoisting이 일어나고 종속성을 최적화시킨다.
   lerna add를 사용할 때는 --scope 옵션을 통해서 어느 패키지에 설치할 것인가를 명시해준다. --scope를 주지 않을 경우 모든 패키지에 설치된다.
   `lerna add commander --scope=log-cli`
   `lerna add chalk --scope=log-core`

8. 각 패키지 기능 추가 후 실행
   `npm link packages/log-cli`
   위 명령어를 실행하니 에러가 발생했다.

   ```
   npm ERR! code EACCES
   npm ERR! syscall symlink
   npm ERR! path /Users/brown/dev/Lerna_Practice/packages/log-cli
   npm ERR! dest /usr/local/lib/node_modules/log-cli
   npm ERR! errno -13
   npm ERR! Error: EACCES: permission denied, symlink '/Users/brown/dev/Lerna_Practice/packages/log-cli' -> '/usr/local/lib/node_modules/log-cli'
   npm ERR!  [Error: EACCES: permission denied, symlink '/Users/brown/dev/Lerna_Practice/packages/log-cli' -> '/usr/local/lib/node_modules/log-cli'] {
   npm ERR!   errno: -13,
   npm ERR!   code: 'EACCES',
   npm ERR!   syscall: 'symlink',
   npm ERR!   path: '/Users/brown/dev/Lerna_Practice/packages/log-cli',
   npm ERR!   dest: '/usr/local/lib/node_modules/log-cli'
   npm ERR! }
   npm ERR!
   npm ERR! The operation was rejected by your operating system.
   npm ERR! It is likely you do not have the permissions to access this file as the current user
   npm ERR!
   npm ERR! If you believe this might be a permissions issue, please double-check the
   npm ERR! permissions of the file and its containing directories, or try running
   npm ERR! the command again as root/Administrator.

   npm ERR! A complete log of this run can be found in:
   npm ERR!     /Users/brown/.npm/_logs/2021-06-16T09_26_04_816Z-debug.log
   ```

   yarn 쓰고 있는데 갑자기 npm 명령어도 이상한데 에러까지 뜨니까 혼란했다.
   이 과정에서 삽질 하다가 어느순간 진행이 되는데
   `lerna link` or `lerna bootstrap` 둘 중 하나가 해결해 준듯 하다.

---

(6/17 추가)
lerna bootstrap 이 실행되면 다음을 수행합니다.

1. 각 패키지의 모든 외부 종속성을 npm install 로 설치
2. 서로 의존하는 Lerna 패키지를 Symlink (심볼릭 링크) 처리
3. 모든 bootstrapped 패키지에서 npm run prepublish 실행 (–ignore-prepublish 가 전달 되지 않는 한)
4. 모든 bootstrapped 패키지에서 npm run prepare 실행
   bootstrap 명령 중에 npm install 에 직접 인수로 전달 될 문자열을 터미널 실행 명령어 뒤에 {옵션} 으로 설정하여 npm 클라이언트에 전달하거나 lerna.json 파일에서 npmClientArgs 속성에 glob 배열형태로 설정할 수 있습니다.
   참조 https://www.js2uix.com/frontend/lerna-step-2/

---

1. 패키지내 참조 및 설치
   위의 과정만으로도 로컬에서는 log-cli 와 log-core 가 연동된다. 하지만 실제 패키지를 배포하면 오류가 발생한다고 한다.
   이 오류는 log-cli에서 log-core를 삽입하였는데 실제로 log-cli에는 log-core가 종속되지 않았기 때문에 발생한다. 이를 해결하기 위해서는 lerna add를 통해서 log-cli에 log-core를 설치해 해줘야 한다.
   `lerna add log-core --scope=log-cli`

2. commit and push
   반영하기 전에 .gitignore를 작성하자
   code .gitignore

   ```
   .idea
   node_modules
   *.lock
   *lock.json
   ```

3. 패키지 배포

   - 배포전에 npm 로그인을 해야한다.
   - 로그인 체크는 npm whoami
   - 배포는 `lerna publish`
   - 배포된 패키지는 72시간이 지나면 삭제할 수 없어서 불필요한 패키지라면 미리 삭제하자.

   이쯤에서 삭제하고 마무리!

---

https://pks2974.medium.com/mono-repo-%EB%A5%BC-%EC%9C%84%ED%95%9C-lerna-%EA%B0%84%EB%8B%A8-%EC%A0%95%EB%A6%AC%ED%95%98%EA%B8%B0-65c22029988

**Lerna Bootstrap**

- lerna 에서 가장 중요한 명령어는 bootstrap 이다.
- lerna bootstrap --hoist 를 실행하면, ignore 로 설정된 package 를 제외한 모든 package 의 npm install 을 실행한다.
- 이때 공통된 module 들은 root 의 node_modules 에 install 하고 각 package 들에 연결된다. 만약 각 package 별로 버전이 다를 경우, warning 메시지를 띄우고, package 의 node_modules 에 install 된다.
- 각 패키지별로 버전이 다른경우를 체크할 수 있으며, 하나의 버전으로 모을 수 있다.
- 가끔 설치가 꼬일수 있는데, 이 경우 lerna clean 으로 모든 package 의 node_modules 를 삭제하고, lerna bootstrap 다시 설치할 수 있다.

package.json 의 dev dependencies
lerna 에서는 제품에 포함되지 않는, dev dependencies 를 root 로 모을 수 있다.
lerna link convert 명령어를 실행하면, 모든 package 에 설정된 devDependencies 을 root devDependencies 으로 옮긴다.
그리고, dependencies 에 package 를 file:packages/package-1 형태로 연결한다.

npm 배포참조
https://kdydesign.github.io/2020/08/28/npm-tutorial/
