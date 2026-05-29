using System;
using System.IO;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Collections.Generic;
using Microsoft.Win32;
using System.Drawing;
using System.Diagnostics;
using System.Linq;
using System.IO.Compression;

namespace Every1Play_Fix
{
    public partial class Form1 : Form
    {
        // =========================
        // UI
        // =========================
        Panel sidebar;
        Panel contentPanel;
		Panel activeUnderline;
        Button installTabButton;
        Button addLibraryTabButton;
        Button fixTabButton;
		Button aboutTabButton;
        Panel installGameTab;
        Panel addLibraryTab;
        Panel fixInternetTab;
		Panel aboutTab;

        Label installTitle;

Panel statusPanel;
Label statusPrefix;
Label statusText;

        Button installBrowseButton;
        Button uninstallButton;

        Label statusLabel;
        ListBox listBox;

        Panel dropPanel;
        Label dropText;
		Label fixDropText;

Panel toastPanel;
Label toastLabel;

Button toastYesButton;
Button toastNoButton;


TaskCompletionSource<bool>? toastWaiter;

        // =========================
        // LOGIC
        // =========================
        int foundCount = 0;
		private class ZipCopyResult
{
    public int LuaCount;

    public int ManifestCount;
}
        HashSet<string> foundPaths =
            new HashSet<string>(
                StringComparer.OrdinalIgnoreCase);

        string installDir =
            Path.Combine(
                Environment.GetFolderPath(
                    Environment.SpecialFolder.LocalApplicationData),
                "Every1Play Shopee");

        // =========================
        // CONSTRUCTOR
        // =========================
        public Form1()
        {
            InitializeComponent();
			this.Icon =
    new Icon(
        Path.Combine(
            AppDomain.CurrentDomain.BaseDirectory,
            "icon.ico"));

            // =========================
            // FORM
            // =========================
            this.Text = "Every1Play Manager V1.0 - 280526";

            this.Width = 780;
            this.Height = 520;

            this.FormBorderStyle =
    FormBorderStyle.FixedDialog;

            this.MaximizeBox = false;

            this.StartPosition =
                FormStartPosition.CenterScreen;

            this.BackColor =
                Color.FromArgb(26, 40, 56);

            this.Font =
                new Font("Bahnschrift", 10);

            // =========================
            // SIDEBAR
            // =========================
            sidebar = new Panel();

            sidebar.Dock = DockStyle.Left;

            sidebar.Width = 220;

            sidebar.BackColor =
                Color.FromArgb(23, 26, 33);
				activeUnderline =
    new Panel();

activeUnderline.Width =
    4;

activeUnderline.Height =
    52;

activeUnderline.BackColor =
    Color.FromArgb(
        102,
        192,
        244);

activeUnderline.Left =
    0;

sidebar.Controls.Add(
    activeUnderline);

// LOGO ICON
PictureBox logo =
    new PictureBox();

logo.Width =
    140;

logo.Height =
    90;

logo.Left =
    40;

logo.Top =
    10;

logo.SizeMode =
    PictureBoxSizeMode.Zoom;

string logoPath =
    Path.Combine(
        AppDomain.CurrentDomain.BaseDirectory,
        "logo.png");

if (File.Exists(logoPath))
{
    logo.Image =
        Image.FromFile(
            logoPath);
}

sidebar.Controls.Add(
    logo);


            // INSTALL TAB BUTTON
            installTabButton =
                CreateSidebarButton(
                    "Install Tool",                    110);

            installTabButton.Click +=
                (s, e) =>
                {
                    ShowTab(
                        installGameTab,
                        installTabButton);
                };
				

            // ADD LIBRARY TAB BUTTON
						
            addLibraryTabButton =
                CreateSidebarButton(
                    "Add Game to Library",
                    170);

            addLibraryTabButton.Click +=
                (s, e) =>
                {
                    ShowTab(
                        addLibraryTab,
                        addLibraryTabButton);
                };

            // FIX INTERNET TAB BUTTON
            fixTabButton =
                CreateSidebarButton(
                    "Fix Download Error",
                    230);

            fixTabButton.Click +=
                (s, e) =>
                {
                    ShowTab(
                        fixInternetTab,
                        fixTabButton);
                };
				
			sidebar.Controls.Add(aboutTabButton);
            sidebar.Controls.Add(fixTabButton);
            sidebar.Controls.Add(addLibraryTabButton);
            sidebar.Controls.Add(installTabButton);
            sidebar.Controls.Add(logo);
	
			// ABOUT TAB BUTTON
aboutTabButton =
    CreateSidebarButton(
        "About Us",
        0);

aboutTabButton.Click +=
    (s, e) =>
    {
        ShowTab(
            aboutTab,
            aboutTabButton);
    };

sidebar.Controls.Add(
    aboutTabButton);

// PLACE BUTTON AT BOTTOM
sidebar.Resize +=
    (s, e) =>
    {
        aboutTabButton.Top =
            sidebar.ClientSize.Height -
            aboutTabButton.Height -
            20;
    };
	
	
            // =========================
            // CONTENT PANEL
            // =========================
            contentPanel = new Panel();

            contentPanel.Dock = DockStyle.Fill;

            contentPanel.BackColor =
                Color.FromArgb(32, 47, 66);

            this.Controls.Add(contentPanel);
            this.Controls.Add(sidebar);
toastPanel =
    new Panel();

toastPanel.Width =
    320;

toastPanel.Height =
    100;

toastPanel.BackColor =
    Color.FromArgb(
        15,
        20,
        30);

toastPanel.Visible =
    false;

toastPanel.Left =
    this.ClientSize.Width -
    toastPanel.Width -
    20;

toastPanel.Top =
    this.ClientSize.Height -
    toastPanel.Height -
    20;

toastLabel =
    new Label();

toastLabel.Width =
    280;

toastLabel.Height =
    50;

toastLabel.Left =
    23;

toastLabel.Top =
    3;

toastLabel.ForeColor =
    Color.White;

toastLabel.TextAlign =
    ContentAlignment.MiddleCenter;

toastPanel.Controls.Add(
    toastLabel);

toastYesButton =
    CreateMainButton(
        "Sure!",
        48,
        50);

toastYesButton.Width =
    100;

toastYesButton.Height =
    35;

toastNoButton =
    CreateMainButton(
        "Nope!",
        168,
        50);

toastNoButton.Width =
    100;

toastNoButton.Height =
    35;

toastYesButton.Click +=
    (s, e) =>
{
    toastWaiter?.TrySetResult(
        true);
};

toastNoButton.Click +=
    (s, e) =>
{
    toastWaiter?.TrySetResult(
        false);
};

toastPanel.Controls.Add(
    toastYesButton);

toastPanel.Controls.Add(
    toastNoButton);

this.Controls.Add(
    toastPanel);

this.Resize +=
    (s, e) =>
{
    toastPanel.Left =
        (this.ClientSize.Width -
        toastPanel.Width) / 2;

    toastPanel.Top =
        (this.ClientSize.Height -
        toastPanel.Height) / 2;
};
            // =========================
            // INSTALL TAB
            // =========================
            installGameTab = new Panel();

            installGameTab.Dock =
                DockStyle.Fill;

            installGameTab.BackColor =
                Color.FromArgb(32, 47, 66);

            installTitle = new Label();

            installTitle.Text =
                "Every1Play Tools Setup";
			
			
            installTitle.Font =
                new Font(
                    "Bahnschrift SemiBold",
                    24);

            installTitle.ForeColor =
                Color.White;

            installTitle.AutoSize = true;

            installTitle.Top = 40;
            installTitle.Left = 97;

            statusPanel =
    new Panel();

statusPanel.Width =
    360;

statusPanel.Height =
    42;

statusPanel.Left =
    86;

statusPanel.Top =
    100;

statusPanel.BackColor =
    Color.FromArgb(
        15,
        20,
        30);

statusPanel.BorderStyle =
    BorderStyle.FixedSingle;


// STATUS: label
statusPrefix =
    new Label();

statusPrefix.Text =
    "STATUS :";

statusPrefix.ForeColor =
    Color.FromArgb(
        102,
        192,
        244);

statusPrefix.Font =
    new Font(
        "Bahnschrift SemiBold",
        13);

statusPrefix.AutoSize =
    true;

statusPrefix.Left =
    15;

statusPrefix.Top =
    9;


// actual message
statusText =
    new Label();

statusText.Text =
    "Ready!";

statusText.ForeColor =
    Color.White;

statusText.Font =
    new Font(
        "Bahnschrift SemiBold",
        13);

statusText.AutoSize =
    true;

statusText.Left =
    95;

statusText.Top =
    9;


statusPanel.Controls.Add(
    statusPrefix);

statusPanel.Controls.Add(
    statusText);

            // INSTALL BUTTON
            installBrowseButton =
                CreateMainButton(
                    "Install Tool",
                    80,
                    160);

            installBrowseButton.Click +=
                InstallBrowseButton_Click;

            // UNINSTALL BUTTON
            uninstallButton =
                CreateMainButton(
                    "Uninstall Tool",
                    280,
                    160);

            uninstallButton.Click +=
                UninstallButton_Click;

			installGameTab.Controls.Add(installTitle);
            installGameTab.Controls.Add(
    statusPanel);
            installGameTab.Controls.Add(installBrowseButton);
            installGameTab.Controls.Add(uninstallButton);
			// RESTART STEAM BUTTON
Button restartSteamTab1Button =
    CreateMainButton(
        "Restart Steam",
        80,
        230);

restartSteamTab1Button.Click +=
    async (s, e) =>
{
    restartSteamTab1Button.Enabled = false;

    statusText.Text =
        "Restarting Steam...";

    try
    {
        foreach (Process proc in
            Process.GetProcessesByName("steam"))
        {
            try
            {
                proc.Kill();
                proc.WaitForExit();
            }
            catch { }
        }

        await Task.Delay(2500);

        string steamExe = "";

        using RegistryKey? key =
            Registry.CurrentUser.OpenSubKey(
                @"Software\Valve\Steam");

        if (key != null)
        {
            object? value =
                key.GetValue("SteamExe");

            if (value != null)
            {
                steamExe =
                    value.ToString() ?? "";
            }
        }

        if (File.Exists(steamExe))
        {
            Process.Start(
                new ProcessStartInfo
                {
                    FileName =
                        steamExe,
                    UseShellExecute =
                        true
                });

            statusText.Text =
    "Steam restarted!";
        }
        else
        {
            statusText.Text =
                "Steam.exe not found!";
        }
    }
    catch
    {
        statusText.Text =
            "Restart failed!";
    }

    restartSteamTab1Button.Enabled =
        true;
};

// EXIT STEAM BUTTON
Button exitSteamButton =
    CreateMainButton(
        "Exit Steam",
        280,
        230);

exitSteamButton.Click +=
    (s, e) =>
{
    int closed = 0;

    foreach (Process proc in
        Process.GetProcessesByName("steam"))
    {
        try
        {
            proc.Kill();
            proc.WaitForExit();

            closed++;
        }
        catch { }
    }

    statusText.Text =
        closed > 0
        ? "Steam closed!"
        : "Steam not running!";
};

installGameTab.Controls.Add(
    restartSteamTab1Button);

installGameTab.Controls.Add(
    exitSteamButton);


            // =========================
            // ADD LIBRARY TAB
            // =========================
            addLibraryTab = new Panel();

            addLibraryTab.Dock =
                DockStyle.Fill;

            addLibraryTab.BackColor =
                Color.FromArgb(32, 47, 66);

            Label addLibraryTitle = new Label();

            addLibraryTitle.Text =
                "Add Game to Steam Library";

            addLibraryTitle.Font =
                new Font(
                    "Bahnschrift SemiBold",
                    22);

            addLibraryTitle.ForeColor =
                Color.White;

            addLibraryTitle.AutoSize = true;

            addLibraryTitle.Top = 35;
            addLibraryTitle.Left = 40;

            Label addLibraryDesc = new Label();

            addLibraryDesc.Text =
                "> Auto Extract and Copy .lua File to Steam/config/lua.";

            addLibraryDesc.Font =
                new Font("Bahnschrift", 10);

            addLibraryDesc.ForeColor =
                Color.Silver;

            addLibraryDesc.AutoSize = true;

            addLibraryDesc.Top = 90;
            addLibraryDesc.Left = 42;

            // =========================
            // DROP PANEL
            // =========================
            dropPanel = new Panel();

            dropPanel.Width = 420;
            dropPanel.Height = 220;

            dropPanel.Left =
                (765 - 220 - dropPanel.Width) / 2;

            dropPanel.Top = 130;

            dropPanel.BackColor =
                Color.FromArgb(15, 20, 30);
				

            dropPanel.AllowDrop = true;

            dropPanel.DragEnter += AddLibrary_DragEnter;
            dropPanel.DragDrop += AddLibrary_DragDrop;



            // DROP TEXT
            dropText = new Label();

            dropText.Text =
                "Drop ZIP Files Here";

            dropText.ForeColor =
                Color.White;

            dropText.Font =
                new Font(
                    "Bahnschrift SemiBold",
                    18);

            dropText.Dock =
                DockStyle.Fill;

            dropText.TextAlign =
                ContentAlignment.MiddleCenter;

            dropText.AllowDrop = true;

            dropText.DragEnter += AddLibrary_DragEnter;
            dropText.DragDrop += AddLibrary_DragDrop;

            dropPanel.Controls.Add(dropText);

            addLibraryTab.Controls.Add(addLibraryTitle);
            addLibraryTab.Controls.Add(addLibraryDesc);
            addLibraryTab.Controls.Add(dropPanel);
			Button gameListButton =
    CreateMainButton(
        "Game List",
        80,
        370);

gameListButton.Click +=
    (s, e) =>
{
    try
    {
        string luaFolder = "";

        using RegistryKey? key =
            Registry.CurrentUser.OpenSubKey(
                @"Software\Valve\Steam");

        if (key != null)
        {
            object? value =
                key.GetValue(
                    "SteamPath");

            if (value != null)
            {
                luaFolder =
                    Path.Combine(
                        value.ToString()!
                        .Replace('/', '\\'),
                        "config",
                        "lua");
            }
        }

        if (!Directory.Exists(luaFolder))
        {
            MessageBox.Show(
                "No Games Installed",
                "Game List",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information);

            return;
        }

        string[] files =
            Directory.GetFiles(
                luaFolder,
                "*.lua");

        Form popup =
            new Form();

        popup.Text =
            "Games Imported";

        popup.Width =
            420;

        popup.Height =
            500;

        popup.StartPosition =
            FormStartPosition.CenterParent;

        popup.BackColor =
            Color.FromArgb(
                32,
                47,
                66);

        popup.FormBorderStyle =
            FormBorderStyle.FixedDialog;

        popup.MaximizeBox =
            false;

        Panel container =
    new Panel();

container.Dock =
    DockStyle.Fill;

container.Padding =
    new Padding(15);

container.BackColor =
    Color.FromArgb(
        32,
        47,
        66);

ListBox gameList =
    new ListBox();

gameList.Dock =
    DockStyle.Fill;

gameList.BackColor =
    Color.FromArgb(
        15,
        20,
        30);

gameList.ForeColor =
    Color.White;

gameList.Font =
    new Font(
        "Bahnschrift",
        11);

gameList.BorderStyle =
    BorderStyle.None;

gameList.HorizontalScrollbar =
    true;

container.Controls.Add(
    gameList);

popup.Controls.Add(
    container);

        gameList.BackColor =
            Color.FromArgb(
                15,
                20,
                30);

        gameList.ForeColor =
            Color.White;

        gameList.Font =
            new Font(
                "Bahnschrift",
                11);

        gameList.BorderStyle =
            BorderStyle.None;

        gameList.HorizontalScrollbar =
            true;

        if (files.Length == 0)
        {
            gameList.Items.Add(
                "No Games Installed");
        }
        else
        {
            foreach (string file in files)
{
    string gameName =
        Path.GetFileNameWithoutExtension(file)
        .Replace(
            " - Every1Play",
            "",
            StringComparison.OrdinalIgnoreCase)
        .Trim();

    gameList.Items.Add(
        gameName);
}
        }

                popup.ShowDialog(
            this);
    }
    catch
    {
        MessageBox.Show(
            "Failed Loading Games");
    }
};

addLibraryTab.Controls.Add(
    gameListButton);
	Button deleteGamesButton =
    CreateMainButton(
        "Delete All Game",
        280,   // right side of Restart Steam
        370);

deleteGamesButton.Click +=
    async (s, e) =>
{
    deleteGamesButton.Enabled = false;

    try
    {
bool confirm =
    await ShowConfirmationToast(
        "Doing this will Remove All Games.\nDrag ZIP to Add Games again!");

if (!confirm)
{
    deleteGamesButton.Enabled =
        true;

    return;
}

        dropText.Text =
            "Deleting Files...";

        await Task.Run(() =>
        {
            string steamPath = "";

            using RegistryKey? key =
                Registry.CurrentUser.OpenSubKey(
                    @"Software\Valve\Steam");

            if (key != null)
            {
                object? value =
                    key.GetValue("SteamPath");

                if (value != null)
                {
                    steamPath =
                        value.ToString()!
                        .Replace('/', '\\');
                }
            }

            if (string.IsNullOrWhiteSpace(
                steamPath))
                return;

            string luaDir =
    Path.Combine(
        steamPath,
        "config",
        "lua");

if (Directory.Exists(luaDir))
{
    foreach (string file in
        Directory.GetFiles(
            luaDir,
            "*.lua"))
    {
        try
        {
            File.Delete(
                file);
        }
        catch { }
    }
}
			string xinputFile =
    Path.Combine(
        steamPath,
        "xinput1_4.dll");

if (File.Exists(
    xinputFile))
{
    try
    {
        File.Delete(
            xinputFile);
    }
    catch { }
}
        });

        dropText.Text =
    "All Games Deleted\n\nRe-import ZIP files to\nadd games again!";

        await Task.Delay(5000);

        dropText.Text =
            "Drop ZIP Files Here";
    }
    catch
    {
        dropText.Text =
            "Delete Failed";
    }

    deleteGamesButton.Enabled = true;
};

addLibraryTab.Controls.Add(
    deleteGamesButton);
	
			// =========================

            // =========================
            // FIX INTERNET TAB
            // =========================
            fixInternetTab = new Panel();

            fixInternetTab.Dock =
                DockStyle.Fill;

            fixInternetTab.BackColor =
                Color.FromArgb(32, 47, 66);

            Label fixTitle = new Label();

            fixTitle.Text =
                "Fix No Internet Error";

            fixTitle.Font =
                new Font(
                    "Bahnschrift SemiBold",
                    22);

            fixTitle.ForeColor =
                Color.White;

            fixTitle.AutoSize = true;

            fixTitle.Top = 35;
            fixTitle.Left = 40;

            statusLabel = new Label();

            statusLabel.Text =
                "Searching Steam/depotcache Folder...";

            statusLabel.Top = 90;
            statusLabel.Left = 42;

            statusLabel.Width = 700;

            statusLabel.ForeColor =
                Color.Silver;

            statusLabel.Font =
                new Font("Bahnschrift", 10);

            // SAME BOX SIZE AS TAB 2
            Panel fixPanel = new Panel();

            fixPanel.Width = 420;
            fixPanel.Height = 220;

            fixPanel.Left =
                (765 - 220 - fixPanel.Width) / 2;

            fixPanel.Top = 130;

            fixPanel.BackColor =
                Color.FromArgb(15, 20, 30);
				fixPanel.AllowDrop = true;

fixPanel.DragEnter +=
    Form1_DragEnter;

fixPanel.DragDrop +=
    Form1_DragDrop;

            // DROP TEXT FOR TAB 3
listBox = new ListBox();

listBox.Visible = false; // keep list storage only

fixDropText =
    new Label();

fixDropText.Text =
    "Drop ZIP Files Here";

fixDropText.ForeColor =
    Color.White;

fixDropText.Font =
    new Font(
        "Bahnschrift SemiBold",
        18);

fixDropText.Dock =
    DockStyle.Fill;

fixDropText.TextAlign =
    ContentAlignment.MiddleCenter;

fixDropText.AllowDrop =
    true;

fixDropText.DragEnter +=
    Form1_DragEnter;

fixDropText.DragDrop +=
    Form1_DragDrop;

fixPanel.Controls.Add(
    fixDropText);
listBox.AllowDrop = true;

listBox.DragEnter +=
    Form1_DragEnter;

listBox.DragDrop +=
    Form1_DragDrop;
            fixInternetTab.Controls.Add(fixTitle);
            fixInternetTab.Controls.Add(statusLabel);
            fixInternetTab.Controls.Add(fixPanel);

            // DRAG DROP

// =========================
// ABOUT TAB
// =========================
aboutTab =
    new Panel();

aboutTab.Dock =
    DockStyle.Fill;

aboutTab.BackColor =
    Color.FromArgb(
        32,
        47,
        66);

// TITLE
Label aboutTitle =
    new Label();

aboutTitle.Text =
    "About Every1Play Manager";

aboutTitle.Font =
    new Font(
        "Bahnschrift SemiBold",
        24);

aboutTitle.ForeColor =
    Color.White;

aboutTitle.Width =
    500;

aboutTitle.Height =
    50;

aboutTitle.Top =
    20;

aboutTitle.Left =
    (770 - 220 - aboutTitle.Width) / 2;

aboutTitle.TextAlign =
    ContentAlignment.MiddleCenter;


// INFO
Label aboutInfo =
    new Label();

aboutInfo.Text =
	"\nEvery1Play just made this App to automate the whole process.\nBuilt entirely with code generated by ChatGPT, he he :D" +
	"\n\nWhat this does is copying .lua file to steam>config>lua\nso the game is shown in Library, not added to the Account!" +
	"\n\nThat is why it only show on One PC, not Every PC, you need to re-add Zip!" +
	"\n\nMain Tools from Open Steam Tools on Github, open source!\nFeel free to check for yourself!" +
"\n\nHave any Ideas? Found Bugs? Contact Us and Help Improve this apps!" +
"\n\nVersion 1.0 - 28 May 2026";


// =========================
// FOLLOW US LINK
// =========================
LinkLabel aboutLink =
    new LinkLabel();

aboutLink.Text =
    "Follow Us!";

aboutLink.AutoSize =
    true;

aboutLink.Font =
    new Font(
        "Bahnschrift",
        11);

aboutLink.LinkBehavior =
    LinkBehavior.NeverUnderline;

aboutLink.LinkColor =
    Color.FromArgb(
        102,
        192,
        244);

aboutLink.ActiveLinkColor =
    Color.FromArgb(
        102,
        192,
        244);

aboutLink.VisitedLinkColor =
    Color.FromArgb(
        102,
        192,
        244);


// =========================
// VIDEO LINK
// =========================
LinkLabel tutorialLink =
    new LinkLabel();

tutorialLink.Text =
    "Video Tutorial";

tutorialLink.AutoSize =
    true;

tutorialLink.Font =
    new Font(
        "Bahnschrift",
        11);

tutorialLink.LinkBehavior =
    LinkBehavior.NeverUnderline;

tutorialLink.LinkColor =
    Color.FromArgb(
        102,
        192,
        244);

tutorialLink.ActiveLinkColor =
    Color.FromArgb(
        102,
        192,
        244);

tutorialLink.VisitedLinkColor =
    Color.FromArgb(
        102,
        192,
        244);


// =========================
// CENTER PANEL
// =========================
Panel linkPanel =
    new Panel();

linkPanel.Width =
    500;

linkPanel.Height =
    40;

linkPanel.Top =
    430;

linkPanel.Left =
    (aboutTab.ClientSize.Width -
    linkPanel.Width) / 2;

linkPanel.BackColor =
    Color.Transparent;


// POSITION INSIDE PANEL
tutorialLink.Left =
    10;

tutorialLink.Top =
    8;

aboutLink.Left =
    410;

aboutLink.Top =
    8;


// CLICK EVENTS
aboutLink.LinkClicked +=
    (s, e) =>
{
    Process.Start(
        new ProcessStartInfo
        {
            FileName =
                "https://lynk.id/every1play",
            UseShellExecute =
                true
        });
};

tutorialLink.LinkClicked +=
    (s, e) =>
{
    Process.Start(
        new ProcessStartInfo
        {
            FileName ="https://canva.link/z5arl5o4zz5duw9",
            UseShellExecute =
                true
        });
};


// ADD TO PANEL
linkPanel.Controls.Add(
    tutorialLink);

linkPanel.Controls.Add(
    aboutLink);


// ADD PANEL TO TAB
aboutTab.Controls.Add(
    linkPanel);


// KEEP CENTERED
aboutTab.Resize +=
    (s, e) =>
{
    linkPanel.Left =
        (aboutTab.ClientSize.Width -
        linkPanel.Width) / 2;
};

aboutInfo.ForeColor =
    Color.Silver;

aboutInfo.Font =
    new Font(
        "Bahnschrift",
        10);

aboutInfo.Width =
    500;

aboutInfo.Height =
    500;

aboutInfo.Top =
    80;

aboutInfo.Left =
    (770 - 220 - aboutInfo.Width) / 2;

aboutInfo.TextAlign =
    ContentAlignment.TopCenter;

aboutTab.Controls.Add(
    aboutTitle);

aboutTab.Controls.Add(
    aboutInfo);

            // ADD PANELS
            contentPanel.Controls.Add(installGameTab);
            contentPanel.Controls.Add(addLibraryTab);
            contentPanel.Controls.Add(fixInternetTab);
			contentPanel.Controls.Add(aboutTab);

            // DEFAULT TAB
            ShowTab(
                installGameTab,
                installTabButton);

            // START SCAN
            this.Shown += async (s, e) =>
                await StartScan();
        }

        // =========================
        // TAB 2 DRAG ENTER
        // =========================
        private void AddLibrary_DragEnter(
    object? sender,
    DragEventArgs e)
{
    if (e.Data == null)
    {
        e.Effect =
            DragDropEffects.None;

        return;
    }

    string[]? files =
        e.Data.GetData(
            DataFormats.FileDrop)
        as string[];

    if (files == null ||
        files.Length == 0)
    {
        e.Effect =
            DragDropEffects.None;

        return;
    }

    bool onlyZip =
        files.All(file =>
            Path.GetExtension(file)
            .Equals(
                ".zip",
                StringComparison.OrdinalIgnoreCase));

    e.Effect =
        onlyZip
        ? DragDropEffects.Copy
        : DragDropEffects.None;
}

        // =========================
        // TAB 2 DRAG DROP
        // =========================
        private async void AddLibrary_DragDrop(
            object? sender,
            DragEventArgs e)
        {
            try
            {
                if (e.Data == null)
                    return;

                string[]? files =
                    e.Data.GetData(
                        DataFormats.FileDrop)
                    as string[];

                if (files == null)
                    return;

                dropText.Text = "Processing...";

                List<string> importedGames =
    new List<string>();

await Task.Run(() =>
{
    foreach (string file in files)
    {
        if (!file.EndsWith(
            ".zip",
            StringComparison.OrdinalIgnoreCase))
            continue;

        ProcessZipFile(file);

        string gameName =
            Path.GetFileNameWithoutExtension(file)
            .Replace(
                " - Every1Play",
                "",
                StringComparison.OrdinalIgnoreCase)
            .Trim();

        importedGames.Add(
            gameName);
    }
});

dropText.Text =
    "Added:\n" +
    string.Join(
        "\n",
        importedGames.Select(
            x => "• " + x));
            }
            catch
            {
                dropText.Text =
                    "Failed To Process ZIP";
            }

            await Task.Delay(7000);

            dropText.Text =
                "Drop ZIP Files Here";
        }
        private ZipCopyResult ProcessZipFile(
    string zipPath)
{
    ZipCopyResult result =
        new ZipCopyResult();

    try
    {
        string steamPath = "";

        using RegistryKey? key =
            Registry.CurrentUser.OpenSubKey(
                @"Software\Valve\Steam");

        if (key != null)
        {
            object? value =
                key.GetValue("SteamPath");

            if (value != null)
            {
                steamPath =
                    value.ToString()!
                    .Replace('/', '\\');
            }
        }

        if (string.IsNullOrWhiteSpace(
            steamPath))
            return result;

        string tempDir =
            Path.Combine(
                Path.GetTempPath(),
                "Every1Play_" +
                Guid.NewGuid());

        Directory.CreateDirectory(
            tempDir);

        ZipFile.ExtractToDirectory(
            zipPath,
            tempDir);

        string luaDir =
    Path.Combine(
        steamPath,
        "config",
        "lua");

Directory.CreateDirectory(
    luaDir);

        foreach (string luaFile in
    Directory.GetFiles(
        tempDir,
        "*.lua",
        SearchOption.AllDirectories))
{
    File.Copy(
        luaFile,
        Path.Combine(
            luaDir,
            Path.GetFileName(
                luaFile)),
        true);

    result.LuaCount++;
}

        try
        {
            Directory.Delete(
                tempDir,
                true);
        }
        catch { }
    }
    catch { }

    return result;
}

        // =========================
        // MAIN BUTTON STYLE
        // =========================
        private Button CreateMainButton(
            string text,
            int left,
            int top)
        {
            Button btn = new Button();

            btn.Text = text;

            btn.Width = 180;
            btn.Height = 45;

            btn.Left = left;
            btn.Top = top;

            btn.FlatStyle =
                FlatStyle.Flat;

            btn.FlatAppearance.BorderSize = 0;

            btn.BackColor =
                Color.FromArgb(56, 74, 95);

            btn.ForeColor =
                Color.White;

            btn.FlatAppearance.MouseOverBackColor =
                Color.FromArgb(70, 90, 115);

            btn.FlatAppearance.MouseDownBackColor =
                Color.FromArgb(45, 60, 80);

            btn.Cursor =
                Cursors.Hand;

            btn.TabStop = false;

            return btn;
        }

        // =========================
        // SIDEBAR BUTTON STYLE
        // =========================
        private Button CreateSidebarButton(
            string text,
            int top)
        {
            Button btn = new Button();

            btn.Text = text;

            btn.Width = 220;
            btn.Height = 52;

            btn.Left = 0;
            btn.Top = top;

            btn.FlatStyle =
                FlatStyle.Flat;

            btn.FlatAppearance.BorderSize = 0;

            btn.UseVisualStyleBackColor = false;

            btn.BackColor =
                Color.FromArgb(23, 26, 33);

            btn.ForeColor =
                Color.Gainsboro;

            btn.Font =
                new Font(
                    "Bahnschrift SemiBold",
                    11);

            btn.TextAlign =
    ContentAlignment.MiddleLeft;

btn.Padding =
    new Padding(
        25,
        0,
        0,
        0);

btn.Font =
    new Font(
        "Bahnschrift SemiBold",
        11);

            btn.Cursor =
                Cursors.Hand;

            btn.TabStop = false;

            btn.MouseEnter += (s, e) =>
{
    if (btn.ForeColor !=
        Color.FromArgb(102,192,244))
    {
        btn.BackColor =
            Color.FromArgb(
                30,
                35,
                45);
    }
};

btn.MouseLeave += (s, e) =>
{
    if (btn.ForeColor !=
        Color.FromArgb(102,192,244))
    {
        btn.BackColor =
            Color.FromArgb(
                23,
                26,
                33);
    }
};

            return btn;
        }

        // =========================
        // SHOW TAB
        // =========================
        private void ShowTab(
            Panel panel,
            Button activeButton)
        {
            installGameTab.Visible = false;
            addLibraryTab.Visible = false;
            fixInternetTab.Visible = false;
			aboutTab.Visible = true;
			
            Button[] buttons =
{
    installTabButton,
    addLibraryTabButton,
    fixTabButton,
    aboutTabButton
};

foreach (Button btn in buttons)
{
    btn.BackColor =
        Color.FromArgb(
            23,
            26,
            33);

    btn.ForeColor =
        Color.Gainsboro;
}

            addLibraryTabButton.BackColor =
                Color.FromArgb(23, 26, 33);

            fixTabButton.BackColor =
                Color.FromArgb(23, 26, 33);
			
			aboutTabButton.BackColor =
    Color.FromArgb(
        23,
        26,
        33);
            activeButton.ForeColor =
    Color.FromArgb(
        102,
        192,
        244);

activeUnderline.Top =
    activeButton.Top;

activeUnderline.Height =
    activeButton.Height;

activeUnderline.BringToFront();

            panel.Visible = true;

            panel.BringToFront();

            this.ActiveControl = null;
        }

        // =========================
        // FAKE INSTALL UI
        // =========================
        private async Task FakeInstallUI(Label label)
        {
            string[] steps =
            {
                "Preparing Installation...",
                "Importing main dll files...",
                "Import Success...",
                "Finishing Installation..."
            };

            foreach (string step in steps)
            {
                label.Text = step;
                await Task.Delay(800);
            }
        }

        // =========================
        // FAKE UNINSTALL UI
        // =========================
        private async Task FakeUninstallUI(Label label)
        {
            string[] steps =
            {
                "Preparing to Remove...",
                "Deleting Installed Files...",
                "Completing Uninstall..."
            };

            foreach (string step in steps)
            {
                label.Text = step;
                await Task.Delay(700);
            }
        }

        // =========================
        // INSTALL
        // =========================
        private async void InstallBrowseButton_Click(
    object? sender,
    EventArgs e)
{
    try
    {
        installBrowseButton.Enabled = false;

string steamPath = "";

        using RegistryKey? key =
            Registry.CurrentUser.OpenSubKey(
                @"Software\Valve\Steam");

        if (key != null)
        {
            object? value =
                key.GetValue(
                    "SteamPath");

            if (value != null)
            {
                steamPath =
                    value.ToString()!
                    .Replace('/', '\\');
            }
        }

        if (string.IsNullOrWhiteSpace(
            steamPath))
        {
            statusText.Text =
                "Steam folder not found";

            return;
        }
		string[] filesToCheck =
{
    "dwmapi.dll",
    "OpenSteamTool.dll",
    "xinput1_4.dll"
};

bool alreadyInstalled =
    filesToCheck.All(file =>
        File.Exists(
            Path.Combine(
                steamPath,
                file)));

if (alreadyInstalled)
{
    statusText.Text =
        "Tool Already Installed!";

    return;
}

// ONLY SHOW INSTALL ANIMATION
// IF NOT INSTALLED
await FakeInstallUI(
    statusText);

        string sourceFolder =
            AppDomain.CurrentDomain.BaseDirectory;

        string[] filesToCopy =
{
    "dwmapi.dll",
    "OpenSteamTool.dll",
    "xinput1_4.dll"
};

int copied = 0;

        foreach (string file in filesToCopy)
        {
            string sourceFile =
                Path.Combine(
                    sourceFolder,
                    file);

            string destinationFile =
                Path.Combine(
                    steamPath,
                    file);

            if (File.Exists(
                sourceFile))
            {
                File.Copy(
                    sourceFile,
                    destinationFile,
                    true);

                copied++;
            }
        }

        statusText.Text =
            $"Imported {copied}/3 files";
    }
    catch
    {
        statusText.Text =
            "Installation Failed";
    }
    finally
    {
        installBrowseButton.Enabled =
            true;
    }
}

        // =========================
        // UNINSTALL
        // =========================
       private async void UninstallButton_Click(
    object? sender,
    EventArgs e)
{
    try
{
    uninstallButton.Enabled = false;

    string steamPath = "";

    using RegistryKey? key =
        Registry.CurrentUser.OpenSubKey(
            @"Software\Valve\Steam");

    if (key != null)
    {
        object? value =
            key.GetValue(
                "SteamPath");

        if (value != null)
        {
            steamPath =
                value.ToString()!
                .Replace('/', '\\');
        }
    }

    if (string.IsNullOrWhiteSpace(
        steamPath))
    {
        statusText.Text =
            "Steam folder not found";

        return;
    }

    string[] filesToCheck =
    {
        "dwmapi.dll",
        "OpenSteamTool.dll",
        "xinput1_4.dll"
    };

    bool alreadyRemoved =
        filesToCheck.All(file =>
            !File.Exists(
                Path.Combine(
                    steamPath,
                    file)));

    if (alreadyRemoved)
    {
        statusText.Text =
            "Tool Already Uninstalled";

        return;
    }

    if (Process.GetProcessesByName(
            "steam").Length > 0)
    {
        statusText.Text =
            "Exit Steam First";

        return;
    }

    bool confirm =
        await ShowConfirmationToast(
            "Remove installed files?");

    if (!confirm)
    {
        uninstallButton.Enabled =
            true;

        return;
    }

    await FakeUninstallUI(
    statusText);


if (alreadyRemoved)
{
    statusText.Text =
        "Tool Already Uninstalled";

    uninstallButton.Enabled =
        true;

    return;
}

        string[] filesToDelete =
        {
            "dwmapi.dll",
            "OpenSteamTool.dll",
            "xinput1_4.dll"
        };

        int deleted = 0;

        foreach (string file in filesToDelete)
        {
            string fullPath =
                Path.Combine(
                    steamPath,
                    file);

            if (File.Exists(
                fullPath))
            {
                try
                {
                    File.Delete(
                        fullPath);

                    deleted++;
                }
                catch
                {
                }
            }
        }

        statusText.Text =
            $"Removed {deleted}/3 files";
    }
    catch
    {
        statusText.Text =
            "Uninstall Failed";
    }
    finally
    {
        uninstallButton.Enabled =
            true;
    }
}
        // =========================
        // COPY DIRECTORY
        // =========================
        private void CopyDirectory(
            string sourceDir,
            string destinationDir)
        {
            Directory.CreateDirectory(destinationDir);

            foreach (string file in Directory.GetFiles(
                sourceDir,
                "*",
                SearchOption.AllDirectories))
            {
                string relative =
                    Path.GetRelativePath(
                        sourceDir,
                        file);

                string dest =
                    Path.Combine(
                        destinationDir,
                        relative);

                string? dir =
                    Path.GetDirectoryName(dest);

                if (!string.IsNullOrWhiteSpace(dir))
                    Directory.CreateDirectory(dir);

                File.Copy(file, dest, true);
            }
        }
private async Task<bool> ShowConfirmationToast(
    string message)
{
    toastLabel.Text =
        message;

   toastPanel.Left =
    sidebar.Width +
    ((contentPanel.Width -
    toastPanel.Width) / 2);

toastPanel.Top =
    (contentPanel.Height -
    toastPanel.Height) / 2;

toastPanel.Visible =
    true;

toastPanel.BringToFront();

    toastWaiter =
        new TaskCompletionSource<bool>();

    bool result =
        await toastWaiter.Task;

    toastPanel.Visible =
        false;

    return result;
}
        // =========================
        // START SCAN
        // =========================
        private async Task StartScan()
        {
            listBox.Items.Clear();

listBox.Items.Add(
    "Drop ZIP Files Here");

foundPaths.Clear();

foundCount = 0;

            await Task.Run(() =>
            {
                foreach (var path in GetSteamPathsFromRegistry())
                    AddSteamPath(path);

                foreach (DriveInfo drive in DriveInfo.GetDrives())
                {
                    try
                    {
                        if (!drive.IsReady)
                            continue;

                        string root =
                            drive.RootDirectory.FullName;

                        string[] possible =
                        {
                            Path.Combine(root,
                                "Program Files (x86)",
                                "Steam"),

                            Path.Combine(root,
                                "Program Files",
                                "Steam"),

                            Path.Combine(root,
                                "Steam"),

                            Path.Combine(root,
                                "Games",
                                "Steam"),

                            Path.Combine(root,
                                "SteamLibrary",
                                "Steam")
                        };

                        foreach (var p in possible)
                            AddSteamPath(p);
                    }
                    catch { }
                }
            });

            statusLabel.Text =
                foundCount == 0
                ? "> Only for User that get Download Error! No Restart Required!"
                : $"> Only for User that get Download Error! No Restart Required!";
        }

        // =========================
        // ADD STEAM PATH
        // =========================
        private void AddSteamPath(string? steamPath)
        {
            if (string.IsNullOrWhiteSpace(steamPath))
                return;

            try
            {
                steamPath =
                    Path.GetFullPath(steamPath)
                    .Replace('/', '\\')
                    .TrimEnd('\\')
                    .ToLowerInvariant();

                if (!Directory.Exists(steamPath))
                    return;

                string depot =
                    Path.Combine(
                        steamPath,
                        "depotcache");

                depot =
                    Path.GetFullPath(depot)
                    .Replace('/', '\\')
                    .TrimEnd('\\')
                    .ToLowerInvariant();

                if (!Directory.Exists(depot))
                    Directory.CreateDirectory(depot);

                if (!foundPaths.Add(depot))
                    return;

                foundCount++;

            }
            catch { }
        }

        // =========================
        // GET STEAM PATH
        // =========================
        private string?[] GetSteamPathsFromRegistry()
        {
            List<string?> paths =
                new List<string?>();

            try
            {
                using RegistryKey? key =
                    Registry.CurrentUser.OpenSubKey(
                        @"Software\Valve\Steam");

                if (key != null)
                {
                    object? value =
                        key.GetValue("SteamPath");

                    if (value != null)
                    {
                        string path =
                            value.ToString() ?? "";

                        path =
                            path.Replace('/', '\\');

                        paths.Add(path);
                    }
                }
            }
            catch { }

            return paths.ToArray();
        }

        // =========================
        // DRAG ENTER
        // =========================
        private void Form1_DragEnter(
            object? sender,
            DragEventArgs e)
        {
            if (e.Data == null)
            {
                e.Effect =
                    DragDropEffects.None;

                return;
            }

            string[]? files =
                e.Data.GetData(
                    DataFormats.FileDrop)
                as string[];

            if (files != null &&
                files.Length > 0 &&
                files.All(file =>
                    file.EndsWith(
                        ".zip",
                        StringComparison.OrdinalIgnoreCase)))
            {
                e.Effect =
                    DragDropEffects.Copy;
            }
            else
            {
                e.Effect =
                    DragDropEffects.None;
            }
        }

        // =========================
        // DRAG DROP
        // =========================
        private async void Form1_DragDrop(
            object? sender,
            DragEventArgs e)
        {
            if (e.Data == null)
                return;

            string[]? files =
                e.Data.GetData(
                    DataFormats.FileDrop)
                as string[];

            if (files == null)
                return;

            fixDropText.Text =
    "Processing ZIP files...";

            int copiedCount = 0;

List<string> gameNames =
    new List<string>();
            await Task.Run(() =>
            {
                foreach (string zipFile in files)
                {
					string gameName =
    Path.GetFileNameWithoutExtension(
        zipFile);

if (!gameNames.Contains(
    gameName,
    StringComparer.OrdinalIgnoreCase))
{
    gameNames.Add(
        gameName);
}
                    if (!zipFile.EndsWith(
                        ".zip",
                        StringComparison.OrdinalIgnoreCase))
                        continue;

                    string tempFolder =
                        Path.Combine(
                            Path.GetTempPath(),
                            "ManifestTemp_" +
                            Guid.NewGuid());

                    try
                    {
                        Directory.CreateDirectory(
                            tempFolder);

                        ZipFile.ExtractToDirectory(
                            zipFile,
                            tempFolder);

                        string[] manifests =
                            Directory.GetFiles(
                                tempFolder,
                                "*.manifest",
                                SearchOption.AllDirectories);

                        foreach (string manifest in manifests)
                        {
                            foreach (string depotPath in foundPaths)
                            {
                                try
                                {
                                    string destination =
                                        Path.Combine(
                                            depotPath,
                                            Path.GetFileName(
                                                manifest));

                                    File.Copy(
                                        manifest,
                                        destination,
                                        true);

                                    copiedCount++;
                                }
                                catch
                                {
                                }
                            }
                        }
                    }
                    catch
                    {
                    }

                    try
                    {
                        if (Directory.Exists(
                            tempFolder))
                        {
                            Directory.Delete(
                                tempFolder,
                                true);
                        }
                    }
                    catch
                    {
                    }
                }
            });

            if (copiedCount > 0)
{
    string names =
        string.Join(
            ", ",
            gameNames);

    fixDropText.Text =
        $"Fixed {names} \n\nSuccess! Try Downloading Now!";
}
else
{
    fixDropText.Text =
        "No manifest files found";
}
await Task.Delay(7000);

fixDropText.Text =
    "Drop ZIP Files Here";
        }
    }
}
