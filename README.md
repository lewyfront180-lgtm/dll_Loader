# DLL Loader + Plugin System (WPF .NET 4.8)

This project is a simple **DLL Loader (plugin host)** written in WPF (.NET Framework 4.8).

It allows you to load external `.dll` files and execute them dynamically inside the application.

---

##  Features

- Load external DLL plugins at runtime
- Execute plugin entry method (`SquareApp.Start()`)
- Simple plugin architecture
- Example plugin included:
  - File Explorer
  - Create/Delete files and folders
  - Rename files
  - Search system
  - Quick access tabs (Desktop, Documents, etc.)

---

## 📂 How plugins work

Each DLL must contain a class like:

```csharp
public class SquareApp
{
    public static void Start()
    {
        // your app code here
    }
}
