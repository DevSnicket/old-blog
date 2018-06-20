My previous post [The C(G) Word(s)](2014-07-25-the-cg-words.md) described some of the problems involved in code generation (heterogeneous) and the possibility of generating code from code (homogeneous). Doing so is a form of metaprogramming or automated programming. At the fringes of code generation is Visual Studio's Snippets feature. However, this is like IntelliSense in its intended to speed up typing and so does not regenerate. To the opposite side of code generation in .NET is IL generation ([intermediate language](http://msdn.microsoft.com/en-us/library/c5tkafs1(vs.71).aspx)). The ability to write IL using an object-model makes generating via code when building or at runtime an unnecessary step. A publicised form of this are re-writers that run on build such as [PostSharp](http://www.postsharp.net/). Another that's widely used, but seldom noticed are the assemblies generated at run-time by .NET's XML serialisation and optionally for regular expressions.

<table>
<tbody>
<tr>
<td>code while typing
e.g. Visual Studio Snippets</td>
<td>code on save
e.g. Visual Studio T4</td>
<td>IL on build
e.g. PostSharp</td>
<td>IL at runtime
e.g. .NET XML / regex</td>
</tr>
<tr>
<td colspan="2">&lt;- static</td>
<td colspan="2">runtime -&gt;</td>
</tr>
</tbody>
</table>

IL writers and re-writers run after the code has been built (i.e. runtime), but code generation usually happens when a file is saved (i.e. static). An advantage of running on build is that the generators behaviour is final (unless you wrote a re-re-writer!). So developers can't (accidently or not) change it by hand invalidating its conventions and future re-generation. An advantage of generating when saving files is that its clear what the behaviour will be, developers aren't normally expected to decompile or read IL. With code the behaviour can be debugged easily and is also checked into source control.

In order to exploit all available advantages my approach is to generate when saving the code, but then regenerate when building. The regenerated output is compared to the file and if not as expected a build error is raised. That puts the onus on the developer to understand how the discrepancy occurred and for them to resolve it (e.g. by resaving).
