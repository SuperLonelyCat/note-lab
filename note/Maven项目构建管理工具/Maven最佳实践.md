### Maven使用

#### 1 mvn clean complie：编译

**先执行 clean，清除 class 文件输出目录 target 中所有内容，然后再进行编译，并将编译的文件放到 target 目录**

#### 2 mvn clean test：测试

**执行工程中所有的单元测试，一般使用 junit测试。需要注意的是，在执行 test 之前会先执行 compile**

#### 3 mvn clean package：打包

**一般用于在开发完成后，在 target 目录下生成可执行的 jar 包（war 包或者供其它项目调用的 jar 包）。需要注意的是，在执行 package 之前会先依次执行 compile 和 test**

#### 4 mvn clean install：安装

**项目打包完成后，可执行的 jar 包（war 包或者供其他项目调用的 jar 包）会输出在 target 目录下，而不是 maven 仓库，如果其他项目要引用，就没有那么方便。使用 install 之后，maven 就会把生成的可执行 jar 包（war 包或者供其它项目调用的 jar 包）安装到 maven 本地仓库中。需要注意的是，执行 install 之前会依次执行 compile、test 以及 package，所以实际上如果是要把项目打包放入到 maven 本地仓库，那么只需要执行 install**

#### 5 mvn clean deploy：发布

**mvn clean install 指令执行完成后只会将生成的可执行 jar 包（war 包或者供其它项目调用的 jar 包）安装到maven本地仓库中，而 mvn clean deploy 不只将可执行 jar 包（war 包或者供其它项目调用的 jar 包）安装到 maven 本地仓库，还会将其部署到 maven 远程私有仓库**

