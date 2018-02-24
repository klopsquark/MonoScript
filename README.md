# MonoScript
MonoScript is a library for .NET that allows you to use C# as if it were a scripting language like Lua, Squirrel, AngelScript, etc.

It achieves this by embedding the Mono C# compiler (MCS), and providing an API to work with it.

MonoScript is designed with sandboxing in mind, in order to make it more approachable for usage within games (its intended use-case). As a result, you have complete control over what types and namespaces are made accessible to scripts.

## Compiling
In order to compile MonoScript, you will first need to compile `jay`. It should be as simple as building the included Premake project, refer to the Premake documentation for information on how to do so.

Once `jay` has been compiled, simply open the MonoScript solution and build it.

Note on VS2017

1. Install `Windows 8.1 SDK and UCRT SDK`
2. Execute `premake5 vs2017` in repo root folder
3. Open and compile `jay.sln`
4. Open and compile `MonoScript.sln`
5. Run demo application

## Usage
Using MonoScript is very simple.

1. Create an instance of the `ScriptBuilder` class and call `Start()` on it.
2. Import any types, references and namespaces you wish to make accessible to scripts.
3. Build the module with `var success = builder.Build(out var module);`
4. Use `module.GetTypes()` to retrieve the script types.

```CSharp
using System;

namespace MonoScript.Sample
{
    public struct MyStruct
    {
        public string Message;

        public override string ToString()
        {
            return Message;
        }
    }
    
    internal class Program
    {
        public static void Main(string[] args)
        {
            var builder = new ScriptBuilder();
            builder.Start();

            builder.ImportType(typeof(Console));
            builder.ImportType<MyStruct>();
            
            builder.AddFromString("Example.cs", @"
                using System;
                using MonoScript.Sample;
                
                public class MyClass
                {
                    public MyClass()
                    {
                        var myStruct = new MyStruct
                        {
                            Message = ""Hello, world!""
                        };
                
                        Console.WriteLine(myStruct);
                    }
                }");

            if (!builder.Build(out var module))
                Console.WriteLine("Failed. {0} error(s). {1} warning(s).", module.ErrorCount, module.WarningCount);
            else
            {
                Console.WriteLine("Succeeded. {0} warning(s).", module.WarningCount);

                foreach (var type in module.GetTypes())
                    Activator.CreateInstance(type);
            }
            
            Console.WriteLine("Press any key to exit . . .");
            Console.ReadKey();
        }
    }
}
```

## Unity
MonoScript has been tested under and confirmed to work with the experimental .NET 4.6 target for Unity projects found in Unity 2017.1 and above.

## License
MonoScript is licensed under the MIT license.
