---
title: WebAssembly 概念
slug: WebAssembly/Concepts
tags:
  - C
  - C++
  - Emscripten
  - JavaScript
  - WebAssembly
  - 文本格式
  - 概念
  - 网络平台
translation_of: WebAssembly/Concepts
---
<p>{{WebAssemblySidebar}}</p>

<p>本文解释了 WebAssembly 如何工作的概念，包括它的目标、它解决的问题以及它是如何在网络浏览器的渲染引擎中运行的。</p>

<h2 id="WebAssembly是什么？">WebAssembly 是什么？</h2>

<p>WebAssembly 是一种运行在现代网络浏览器中的新型代码，并且提供新的性能特性和效果。它设计的目的不是为了手写代码而是为诸如 C、C++和 Rust 等低级源语言提供一个高效的编译目标。</p>

<p>对于网络平台而言，这具有巨大的意义——这为客户端 app 提供了一种在网络平台以接近本地速度的方式运行多种语言编写的代码的方式；在这之前，客户端 app 是不可能做到的。</p>

<p>而且，你在不知道如何编写 WebAssembly 代码的情况下就可以使用它。WebAssembly 的模块可以被导入的到一个网络 app（或 Node.js）中，并且暴露出供 JavaScript 使用的 WebAssembly 函数。JavaScript 框架不但可以使用 WebAssembly 获得巨大性能优势和新特性，而且还能使得各种功能保持对网络开发者的易用性。</p>

<h2 id="WebAssembly的目标">WebAssembly 的目标</h2>

<p>作为 <a href="https://www.w3.org/community/webassembly/">W3C WebAssembly Community Group</a>中的一项开放标准，WebAssembly 是为下列目标而生的：</p>

<ul>
 <li>快速、高效、可移植——通过利用<a href="http://webassembly.org/docs/portability/#assumptions-for-efficient-execution">常见的硬件能力</a>，WebAssembly 代码在不同平台上能够以接近本地速度运行。</li>
 <li>可读、可调试——WebAssembly 是一门低阶语言，但是它有确实有一种人类可读的文本格式（其标准即将得到最终版本），这允许通过手工来写代码，看代码以及调试代码。</li>
 <li>保持安全——WebAssembly 被限制运行在一个安全的沙箱执行环境中。像其他网络代码一样，它遵循浏览器的同源策略和授权策略。</li>
 <li>不破坏网络——WebAssembly 的设计原则是与其他网络技术和谐共处并保持向后兼容。</li>
</ul>

<div class="note">
<p><strong>备注：</strong> WebAssembly 也用在网络和 JavaScript 环境之外（参考<a href="http://webassembly.org/docs/non-web/">非网络嵌入</a>）。</p>
</div>

<h2 id="WebAssembly如何适应网络平台？">WebAssembly 如何适应网络平台？</h2>

<p>网络平台可以被想象成拥有两个部分：</p>

<ul>
 <li>一个运行网络程序（Web app）代码——比如，给你的程序提供能力的 JavaScript——的虚拟机</li>
 <li>一系列网络程序能够调用从而控制网络浏览器/设备功能，并且能够让事物发生变化的网络API（<a href="/zh-CN/docs/Web/API/Document_Object_Model">DOM</a>、<a href="/zh-CN/docs/Web/API/CSS_Object_Model">CSSOM</a>、<a href="/zh-CN/docs/Web/API/WebGL_API">WebGL</a>、<a href="/zh-CN/docs/Web/API/IndexedDB_API">IndexedDB</a>、<a href="/zh-CN/docs/Web/API/Web_Audio_API">Web Audio API</a>等）。</li>
</ul>

<p>从历史角度讲，虚拟机过去只能加载 JavaScript。这对我们而言足够了，因为 JavaScript 足够强大从而能够解决人们在当今网络上遇到的绝大部分问题。尽管如此，当试图把 JavaScript 应用到诸如 3D 游戏、虚拟现实、增强现实、计算机视觉、图像/视频编辑以及大量的要求原生性能的其他领域的时候，我们遇到了性能问题（参考 <a href="http://webassembly.org/docs/use-cases/">WebAssembly 使用案例</a> 获取更多细节）。</p>

<p>而且，下载、解析以及编译巨大的 JavaScript 应用程序的成本是过高的。移动平台和其他资源受限平台进一步放大了这些性能瓶颈。</p>

<p>WebAssembly 是一门不同于 JavaScript 的语言，但是，它不是用来取代 JavaScript 的。相反，它被设计为和 JavaScript 一起协同工作，从而使得网络开发者能够利用两种语言的优势：</p>

<ul>
 <li>JavaScript 是一门高级语言。对于写网络应用程序而言，它足够灵活且富有表达力。它有许多优势——它是动态类型的，不需要编译环节以及一个巨大的能够提供强大框架、库和其他工具的生态系统。</li>
 <li>WebAssembly 是一门低级的类汇编语言。它有一种紧凑的二进制格式，使其能够以接近原生性能的速度运行，并且为诸如 C++和 Rust 等拥有低级的内存模型语言提供了一个编译目标以便它们能够在网络上运行。（注意，WebAssembly 有一个在将来支持使用了垃圾回收内存模型的语言的高级目标。）</li>
</ul>

<p>随着 WebAssembly 出现在了浏览器中，我们前面提到的虚拟机将会加载和运行两种类型的代码——JavaScript 和 WebAssembly。</p>

<p>不同类型的代码能够按照需要进行相互调用——WebAssembly 的 JavaScript API 使用能够被正常调用的 JavaScript 函数封装了导出的 WebAssembly 代码，并且 WebAssembly 代码能够导入和同步调用常规的 JavaScript 函数。事实上，WebAssembly 代码的基本单元被称作一个模块，并且 WebAssembly 的模块在很多方面都和 ES2015 的模块是等价的。</p>

<h3 id="WebAssembly关键概念">WebAssembly 关键概念</h3>

<p>为了理解 WebAssembly 如何在浏览器中运行，需要了解几个关键概念。所有这些概念都是一一映射到了<a href="/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly">WebAssembly 的 JavaScript API</a>中。</p>

<ul>
 <li><strong>模块</strong>：表示一个已经被浏览器编译为可执行机器码的 WebAssembly 二进制代码。一个模块是无状态的，并且像一个<a href="/zh-CN/docs/Web/API/Blob">二进制大对象</a>（Blob）一样能够<a href="/zh-CN/docs/WebAssembly/Caching_modules">被缓存到 IndexedDB</a>中或者在 windows 和 workers 之间进行共享（通过<a href="/zh-CN/docs/Web/API/MessagePort/postMessage">postMessage()</a>函数）。一个模块能够像一个 ES2015 的模块一样声明导入和导出。</li>
 <li><strong>内存</strong>：ArrayBuffer，大小可变。本质上是连续的字节数组，WebAssembly 的低级内存存取指令可以对它进行读写操作。</li>
 <li><strong>表格</strong>：带类型数组，大小可变。表格中的项存储了<strong>不能作为原始字节存储在内存里的对象</strong>的引用（为了安全和可移植性的原因）。</li>
 <li><strong>实例</strong>：一个模块及其在运行时使用的所有状态，包括内存、表格和一系列导入值。一个实例就像一个已经被加载到一个拥有一组特定导入的特定的全局变量的 ES2015 模块。</li>
</ul>

<p>JavaScript API 为开发者提供了创建模块、内存、表格和实例的能力。给定一个 WebAssembly 实例，JavaScript 代码能够调用普通 JavaScript 函数暴露出来的导出代码。通过把 JavaScript 函数导入到 WebAssembly 实例中，任意的 JavaScript 函数都能被 WebAssembly 代码同步调用。</p>

<p>因为 JavaScript 能够完全控制 WebAssembly 代码如何下载、编译运行，所以，JavaScript 开发者甚至可以把 WebAssembly 想象成一个高效地生成高性能函数的 JavaScript 特性。</p>

<p>将来，WebAssembly 模块将会<a href="http://webassembly.org/docs/modules/#integration-with-es6-modules">像 ES2015 模块那样加载</a>（使用&lt;script type='module'&gt;)，这也就意味着 JavaScript 代码能够像轻松地使用一个 ES2015 模块那样来获取、编译和导入一个 WebAssembly 模块。</p>

<h2 id="如何在我的app中使用WebAssembly？">如何在我的 app 中使用 WebAssembly？</h2>

<p>上面我们讨论了 WebAssembly 向网络平台增加的基本要素：</p>

<ul>
 <li>代码的二进制格式</li>
 <li>加载运行该二进制代码的 API</li>
</ul>

<p>现在让我们讨论如何在实践中使用这些基本要素。</p>

<p>WebAssembly 生态系统处在初始阶段；更多的工具会毫无疑问得不断出现。当前，有四个主要的着手点：</p>

<ul>
 <li>使用Emscripten移植一个C/C++应用程序。</li>
 <li>直接在汇编层，编写或生成 WebAssembly 代码。</li>
 <li>编写 Rust 程序，将 WebAssembly 作为它的输出。</li>
 <li>使用<a href="https://www.assemblyscript.org/">AssemblyScript</a>，它类似于 TypeScript 并且可编译成二进制 WebAssmebly 代码</li>
</ul>

<p>让我们讨论这几项：</p>

<h3 id="从CC移植">从C/C++移植</h3>

<p>创建 WASM 代码的众多选项中有两个是在线 WASM 汇编程序或<a href="/zh-CN/docs/Mozilla/Projects/Emscripten">Emscripten</a>。有许多在线 WASM 汇编程序可供选择，例如：</p>

<ul>
 <li><a href="https://wasdk.github.io/WasmFiddle/">WasmFiddle</a></li>
 <li><a href="https://anonyco.github.io/WasmFiddle/">WasmFiddle++</a></li>
 <li><a href="https://mbebenita.github.io/WasmExplorer/">WasmExplorer</a></li>
</ul>

<p>对于那些想知道从哪里开始的人来说，这些是很好的资源，但是他们缺少一些 Emscripten 的工具和优化。</p>

<p>Emscripten工具能够将一段C/C++代码，编译出：</p>

<ul>
 <li>一个.wasm 模块</li>
 <li>用来加载和运行该模块的 JavaScript”胶水“代码</li>
 <li>一个用来展示代码运行结果的 HTML 文档</li>
</ul>

<p><img src="emscripten-diagram.png"></p>

<p>简而言之，工作流程如下所示：</p>

<ol>
 <li>Emscripten首先把C/C++提供给clang+LLVM——一个成熟的开源C/C++编译器工具链，比如，在OSX上是XCode的一部分。</li>
 <li>Emscripten 将 clang+LLVM 编译的结果转换为一个.wasm 二进制文件。</li>
 <li>就自身而言，WebAssembly 当前不能直接的存取 DOM；它只能调用 JavaScript，并且只能传入整形和浮点型的原始数据类型作为参数。这就是说，为了使用任何 Web API，WebAssembly 需要调用到 JavaScript，然后由 JavaScript 调用 Web API。因此，Emscripten 创建了 HTML 和 JavaScript 胶水代码以便完成这些功能。</li>
</ol>

<div class="note">
<p><strong>备注：</strong> 计划将来<a href="https://github.com/WebAssembly/gc/blob/master/README.md">允许 WebAssembly 直接调用 Web API</a>。</p>
</div>

<p>JavaScript 胶水代码并不是像你想象的那么简单。首先，Emscripten 实现了流行的 C/C++库，比如， <a href="https://en.wikipedia.org/wiki/Simple_DirectMedia_Layer">SDL</a>、 <a href="https://en.wikipedia.org/wiki/OpenGL">OpenGL</a>、 <a href="https://en.wikipedia.org/wiki/OpenAL">OpenAL</a>以及部分<a href="https://en.wikipedia.org/wiki/POSIX">POSIX</a>。这些库以 Web API 的形式实现，并且每个库需要一个 JavaScript 胶水代码来连接 WebAssembly 和低层的 Web API。</p>

<p>因此，部分胶水代码实现了 C/C++代码使用的库的功能。而且，胶水代码还包括调用前面提到的 WebAssembly 的 JavaScript API 来获取、加载和运行.wasm 文件的逻辑。</p>

<p>生成的 HTML 文档加载 JavaScript 胶水文件，并且将 stdout 输出到{{htmlelement("textarea")}}。如果应用程序使用了 OpenGL，HTML 文档还会包含一个用来渲染目标的{{htmlelement("canvas")}}标签。</p>

<p>修改 Emscripten 的输出文件并将其转换为需要的 web app 是很容易的。</p>

<p>你可以在<a href="http://emscripten.org">emscripten.org</a>找到关于 Emscripten 的完整文档以及在<a href="/zh-CN/docs/WebAssembly/C_to_wasm">Compiling from C/C++ to WebAssembly</a>找到一个实现工具链并交叉编译你自己的C/C++应用程序为wasm的指南。</p>

<h3 id="直接编写WebAssembly代码">直接编写 WebAssembly 代码</h3>

<p>你想构建自己的编译器，或者你自己的工具，或者创建一个能够在运行时生成 WebAssembly 代码的 JavaScript 库？</p>

<p>就像真实的汇编语言一样，WebAssembly 的二进制格式也有文本表示——两者之间 1:1 对应。你可以手工书写或者生成这种格式然后使用这些工具（<a href="http://webassembly.org/getting-started/advanced-tools/">WebAssemby text-to-binary tools</a>）中的任何一个把它转换为二进制格式。</p>

<p>这有一份如何做这些的简单指南，参考我们的文章<a href="/zh-CN/docs/WebAssembly/Text_format_to_wasm">转换 WebAssembly 文本格式为 wasm</a>。</p>

<h2 id="总结">总结</h2>

<p>本文给你解释了 WebAssembly 是什么，它为什么如此有用，它是如何适应网络的以及你如何才能使用它。</p>

<h2 id="参考">参考</h2>

<ul>
 <li><a href="https://hacks.mozilla.org/category/webassembly/">Mozilla Hacks 博客上关于 WebAssembly 的文章</a></li>
 <li><a href="https://research.mozilla.org/webassembly/">Mozilla Research 上关于 WebAssembly 的文章</a></li>
 <li><a href="/zh-CN/docs/WebAssembly/Loading_and_running">加载运行 WebAssembly 代码</a>——探究如何把你自己的 WebAssembly 模块加载到网页中。</li>
 <li><a href="/zh-CN/docs/WebAssembly/Using_the_JavaScript_API">使用 WebAssembly 的 JavaScript API</a>——探究如何使用 WebAssembly 的 JavaScript API 的其他主要特性。</li>
</ul>