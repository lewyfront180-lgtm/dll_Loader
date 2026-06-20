to create a dll application for this program, you need to have Visual Studio and work in C# and .NET 4.8, set it to x64, example (file explorer)
------------------------------------------------------------------------------------------------------------------------------------------------
public class SquareApp
{
    private static Window win;
    private static ListBox list;
    private static TextBox pathBox;
    private static TextBox searchBox;

    private static string currentPath;

    public static void Start()
    {
        currentPath = Environment.GetFolderPath(Environment.SpecialFolder.UserProfile);

        win = new Window
        {
            Title = "fil",
            Width = 950,
            Height = 650,
            Background = Brushes.White
        };

        BuildUI();

        win.Show();
        LoadFiles();
    }

    // ================= UI =================
    private static void BuildUI()
    {
        var grid = new Grid { Margin = new Thickness(10) };

        grid.RowDefinitions.Add(new RowDefinition { Height = GridLength.Auto }); // tabs
        grid.RowDefinitions.Add(new RowDefinition { Height = GridLength.Auto }); // path
        grid.RowDefinitions.Add(new RowDefinition { Height = GridLength.Auto }); // search
        grid.RowDefinitions.Add(new RowDefinition());                             // list
        grid.RowDefinitions.Add(new RowDefinition { Height = GridLength.Auto }); // buttons

        // ================= TABS =================
        var tabs = new StackPanel
        {
            Orientation = Orientation.Horizontal,
            HorizontalAlignment = HorizontalAlignment.Center,
            Margin = new Thickness(0, 0, 0, 10)
        };

        tabs.Children.Add(TabBtn("Desktop", Environment.SpecialFolder.Desktop));
        tabs.Children.Add(TabBtn("Documents", Environment.SpecialFolder.MyDocuments));
        tabs.Children.Add(TabBtn("Pictures", Environment.SpecialFolder.MyPictures));
        tabs.Children.Add(TabBtn("Videos", Environment.SpecialFolder.MyVideos));

        tabs.Children.Add(Btn("Downloads", () =>
        {
            currentPath = Path.Combine(
                Environment.GetFolderPath(Environment.SpecialFolder.UserProfile),
                "Downloads"
            );

            pathBox.Text = currentPath;
            LoadFiles();
        }));

        Grid.SetRow(tabs, 0);
        grid.Children.Add(tabs);

        // ================= PATH =================
        pathBox = new TextBox
        {
            Text = currentPath,
            Height = 28,
            Margin = new Thickness(0, 0, 0, 5)
        };

        pathBox.KeyDown += (s, e) =>
        {
            if (e.Key.ToString() == "Return" && Directory.Exists(pathBox.Text))
            {
                currentPath = pathBox.Text;
                LoadFiles();
            }
        };

        Grid.SetRow(pathBox, 1);
        grid.Children.Add(pathBox);

        // ================= SEARCH =================
        searchBox = new TextBox
        {
            Height = 28,
            Margin = new Thickness(0, 0, 0, 5)
        };

        searchBox.TextChanged += (s, e) => LoadFiles();

        Grid.SetRow(searchBox, 2);
        grid.Children.Add(searchBox);

        // ================= FILE LIST =================
        list = new ListBox
        {
            SelectionMode = SelectionMode.Extended
        };

        list.MouseDoubleClick += (s, e) =>
        {
            if (list.SelectedItem == null) return;

            string path = list.SelectedItem.ToString();

            if (Directory.Exists(path))
            {
                currentPath = path;
                pathBox.Text = path;
                LoadFiles();
            }
        };

        Grid.SetRow(list, 3);
        grid.Children.Add(list);

        // ================= BUTTONS =================
        var panel = new StackPanel
        {
            Orientation = Orientation.Horizontal,
            HorizontalAlignment = HorizontalAlignment.Center
        };

        panel.Children.Add(Btn("Folder", CreateFolder));
        panel.Children.Add(Btn("TXT", CreateTxt));
        panel.Children.Add(Btn("Delete", DeleteSelected));
        panel.Children.Add(Btn("Rename", RenameSelected));
        panel.Children.Add(Btn("Refresh", LoadFiles));
        panel.Children.Add(Btn("Up", GoUp));

        Grid.SetRow(panel, 4);
        grid.Children.Add(panel);

        win.Content = grid;
    }

    // ================= BUTTON =================
    private static Button Btn(string text, Action action)
    {
        var b = new Button
        {
            Content = text,
            Margin = new Thickness(5),
            Padding = new Thickness(10)
        };

        b.Click += (s, e) => action();
        return b;
    }

    // ================= TAB BUTTON =================
    private static Button TabBtn(string name, Environment.SpecialFolder folder)
    {
        return Btn(name, () =>
        {
            currentPath = Environment.GetFolderPath(folder);
            pathBox.Text = currentPath;
            LoadFiles();
        });
    }

    // ================= LOAD =================
    private static void LoadFiles()
    {
        if (list == null || string.IsNullOrEmpty(currentPath)) return;
        if (!Directory.Exists(currentPath)) return;

        list.Items.Clear();

        string filter = searchBox.Text?.ToLower() ?? "";

        foreach (var dir in Directory.GetDirectories(currentPath))
            if (dir.ToLower().Contains(filter))
                list.Items.Add(dir);

        foreach (var file in Directory.GetFiles(currentPath))
            if (file.ToLower().Contains(filter))
                list.Items.Add(file);
    }

    // ================= CREATE =================
    private static void CreateFolder()
    {
        Directory.CreateDirectory(
            Path.Combine(currentPath, "Folder_" + DateTime.Now.Ticks)
        );

        LoadFiles();
    }

    private static void CreateTxt()
    {
        File.WriteAllText(
            Path.Combine(currentPath, "file_" + DateTime.Now.Ticks + ".txt"),
            "new file"
        );

        LoadFiles();
    }

    // ================= DELETE =================
    private static void DeleteSelected()
    {
        foreach (var item in list.SelectedItems.Cast<object>().ToList())
        {
            string path = item.ToString();

            try
            {
                if (Directory.Exists(path))
                    Directory.Delete(path, true);
                else if (File.Exists(path))
                    File.Delete(path);
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }

        LoadFiles();
    }

    // ================= RENAME =================
    private static void RenameSelected()
    {
        if (list.SelectedItem == null) return;

        string oldPath = list.SelectedItem.ToString();

        string newName = SimpleInput("Rename", "New name:", Path.GetFileName(oldPath));
        if (string.IsNullOrWhiteSpace(newName)) return;

        string newPath = Path.Combine(Path.GetDirectoryName(oldPath), newName);

        try
        {
            if (Directory.Exists(oldPath))
                Directory.Move(oldPath, newPath);
            else if (File.Exists(oldPath))
                File.Move(oldPath, newPath);
        }
        catch (Exception ex)
        {
            MessageBox.Show(ex.Message);
        }

        LoadFiles();
    }

    // ================= UP =================
    private static void GoUp()
    {
        var parent = Directory.GetParent(currentPath);
        if (parent != null)
        {
            currentPath = parent.FullName;
            pathBox.Text = currentPath;
            LoadFiles();
        }
    }

    // ================= SIMPLE INPUT =================
    private static string SimpleInput(string title, string text, string defaultValue)
    {
        Window w = new Window
        {
            Title = title,
            Width = 300,
            Height = 150,
            WindowStartupLocation = WindowStartupLocation.CenterScreen
        };

        var stack = new StackPanel { Margin = new Thickness(10) };

        var box = new TextBox { Text = defaultValue };
        string result = null;

        var btn = new Button
        {
            Content = "OK",
            Margin = new Thickness(0, 10, 0, 0)
        };

        btn.Click += (s, e) =>
        {
            result = box.Text;
            w.Close();
        };

        stack.Children.Add(new TextBlock { Text = text });
        stack.Children.Add(box);
        stack.Children.Add(btn);

        w.Content = stack;
        w.ShowDialog();

        return result;
    }
}
