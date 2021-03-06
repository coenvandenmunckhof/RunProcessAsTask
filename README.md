RunProcessAsTask
================

Simple wrapper around [System.Diagnostics.Process](http://msdn.microsoft.com/en-us/library/system.diagnostics.process.aspx) to expose it as a [System.Threading.Tasks.Task](http://msdn.microsoft.com/en-us/library/system.threading.tasks.task.aspx)<[ProcessResults](https://github.com/jamesmanning/RunProcessAsTask/blob/master/src/RunProcessAsTask/ProcessResults.cs)>

Includes support for cancellation, timeout (via cancellation), and exposes the standard output, standard error, exit code, and run time of the process.

To install into your project:

```powershell
PM> Install-Package RunProcessAsTask
```

# Example Usages

## Synchronous, just easier way of grabbing output / error / runtime for the process

```csharp
var processResults = ProcessEx.RunAsync("git.exe", "pull").Result;

Console.WriteLine("Exit code: " + processResults.ExitCode);
Console.WriteLine("Run time: " + processResults.RunTime);

Console.WriteLine("{0} lines of standard output", processResults.StandardOutput.Length);
foreach (var output in processResults.StandardOutput)
{
    Console.WriteLine("Output line: " + output);
}

Console.WriteLine("{0} lines of standard error", processResults.StandardError.Length);
foreach (var error in processResults.StandardError)
{
    Console.WriteLine("Error line: " + error);
}
```

## Provide timeout for running process

```csharp
public async Task RunCommandWithTimeout(string filename, string arguments, TimeSpan timeout)
{
    var cancellationTokenSource = new CancellationTokenSource(timeout);

    var processStartInfo = new ProcessStartInfo
    {
        FileName = filename,
        Arguments = arguments,
    };
    try
    {
        var processResults = await ProcessEx.RunAsync(processStartInfo, cancellationTokenSource.Token);
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("Timeout of {0} hit while trying to run {1} {2}", timeout, filename, arguments);
    }
}
```

## Run multiple commands with dependencies in an async fashion

```csharp
public async Task ShowLastMatchingCommmit(string regex)
{
    var logProcessResults = await ProcessEx.RunAsync("git.exe", "log --pretty=oneline --all -n 1 -G" + regex);
    if (logProcessResults.ExitCode != 0) return;

    var stdoutSplit = logProcessResults.StandardOutput[0].Split(new[] { ' ' }, 2);
    var commitHash = stdoutSplit[0];
    var commitMessage = stdoutSplit[1];
    Console.WriteLine("Last commit matching {0} was {1} and had commit message {2}", regex, commitHash, commitMessage);
    var showProcessResults = await ProcessEx.RunAsync("git.exe", "show --pretty=fuller " + commitHash);
    foreach (var stdoutLine in showProcessResults.StandardOutput)
    {
        Console.WriteLine("git show output: " + stdoutLine);
    }
}
```
