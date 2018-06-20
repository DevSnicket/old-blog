I’ve been working on a static analysis and reporting system, part of which runs our FxCop rule set. Although FxCop’s days are numbered because of Roslyn we can’t change over until the 200+ rules we run are available. Most examples I’ve found to do this execute its process and parse the results. Looking at it (and other Visual Studio analysis features) I’ve found its internals offer a more flexible, richer and accurate analysis.

Unfortunately when accessing it programmatically I realised it leaks memory after a few projects (we have 800+). Running in a separate app domain wasn’t enough as part of it is unmanaged code. I tried a few approaches to isolate the main process by spawning a sub-process and I think what I’ve got now is reliable.

To use it a delegate and parameters are passed into a method. The method passed in must be static and the parameter must be binary serialisable.

``` csharp
TResult InvokeInProcess<TParameter, TResult>(
    Func<TParameter, TResult> method,
    TParameter parameter)
```

I’m using a ProcessStartInfo with a single fixed argument to recognise the sub-process execution, reusing the executable, redirecting the standard input and output, and setting UseShellExecute so there’s no new window.

``` csharp
new ProcessStartInfo(
    arguments: "RunSubprocess",
    fileName: Assembly.GetExecutingAssembly().Location)
{
    RedirectStandardInput = true,
    RedirectStandardOutput = true,
    UseShellExecute = false,
};
```

The parameter and the MethodInfo of the delegate are then serialised to the standard input's base stream.

``` csharp
new BinaryFormatter().Serialize(
    process.StandardInput.BaseStream,
    new MethodAndParameter
    {
        Method = method.Method,
        Parameter = parameter,
    });
```

The "startup object" (e.g. Program.Main) needs to handle getting executed as a sub-process. It recognises this by checking for the single argument passed in when the main process started it. The input and output streams can then be used either side of an invoke.

``` csharp
if (arguments.Count == 1 &amp;&amp; arguments.First() == "RunSubprocess")
{
    MethodAndParameter methodAndParameter;

    using (Stream input = Console.OpenStandardInput())
        methodAndParameter = (MethodAndParameter)new BinaryFormatter().Deserialize(input);

    using (Stream output = Console.OpenStandardOutput())
        new BinaryFormatter().Serialize(
            output,
            methodAndParameter.Method.Invoke(null, new[] { methodAndParameter.Parameter, }));
}
```

Back on the main process I had some problems with WaitForExit locking up serialisation of the return value in the sub-process. So instead I use a MemoryStream and a while statement that checks HasExited. Once this has finished the return value can be deserialised.

``` csharp
using (var output = new MemoryStream())
{
	while (!process.HasExited)
		process.StandardOutput.BaseStream.CopyTo(output);

	output.Position = 0;

	return new BinaryFormatter().Deserialize(output);
}
```

To make debugging the sub-process easier its possible to attach the debugger between starting the it and serialising the parameter.

``` csharp
if (Debugger.IsAttached)
{
	Boolean isAttached = false;

	do
	{
		try
		{
			((EnvDTE.DTE)System.Runtime.InteropServices.Marshal.GetActiveObject("VisualStudio.DTE.12.0"))
			.Debugger
			.LocalProcesses
			.Cast&lt;EnvDTE.Process&gt;()
			.Single(visualStudioProcess =&gt; visualStudioProcess.ProcessID == process.Id)
			.Attach();

			isAttached = true;
		}
		catch (COMException e)
		{
			if (!e.Message.StartsWith("The message filter indicated that the application is busy."))
				throw;
		}
	}
	while (!isAttached);
}
```

The full source code, which includes an example, is available on [GitHub](https://github.com/devsnicket/ProcessSpawning)</a>.