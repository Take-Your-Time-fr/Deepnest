<img src="https://deepnest.io/img/logo-large.png" alt="Deepnest" width="250">  

**Deepnest**: A fast, robust nesting tool for laser cutters and other CNC tools.  

**Download:** https://deepnest.io  
  
Deepnest is a desktop application based on [SVGNest](https://github.com/Jack000/SVGnest).  

  
- new nesting engine with speed critical code written in C
- merges common lines for laser cuts
- support for DXF files (via conversion)
- new path approximation feature for highly complex parts  
  
	
**Deepnest**: 一个快速有效的用于激光切割和其他CNC机床的排样工具。  

Deepnest是基于[SVGNest](https://github.com/Jack000/SVGnest)的桌面版本。  
  
  
* 使用C语言重新编写的排样引擎
* 方便激光切割合并重合线
* 通过转换支持DXF格式
* 对高度复杂的零件实现新的近似方法
- new path approximation feature for highly complex parts

To rebuild the native nesting engine
- npm install --save-dev electron-rebuild
- npm install
- .\node_modules\.bin\electron-rebuild.cmd
- copy contents of build/Release to minkowski/Release
- To package app run electron-packager . deepnest --platform=win32 --arch=x64
