Để tạo một ứng dụng WPF đơn giản sử dụng đoạn mã kiểm tra thiết bị, bạn cần tạo một dự án WPF mới và thêm mã vào file `MainWindow.xaml` và `MainWindow.xaml.cs`. Dưới đây là một ví dụ đơn giản:

1. **MainWindow.xaml:**
```xml
<Window x:Class="YourNamespace.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Device Checker" Height="350" Width="500">
    <Grid>
        <ListBox Name="DeviceListBox" Margin="10" DisplayMemberPath="DeviceId">
            <ListBox.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal">
                        <TextBlock Text="{Binding DeviceId}" Width="100"/>
                        <TextBlock Text="{Binding IsConnected}" Width="80"/>
                    </StackPanel>
                </DataTemplate>
            </ListBox.ItemTemplate>
        </ListBox>
    </Grid>
</Window>
```

2. **MainWindow.xaml.cs:**
```csharp
using System;
using System.Linq;
using System.Timers;
using System.Windows;

namespace YourNamespace
{
    public partial class MainWindow : Window
    {
        private System.Timers.Timer timer;
        private MainViewModel mainVM;

        public MainWindow()
        {
            InitializeComponent();

            // Initialize ViewModel and timer
            mainVM = new MainViewModel(); // Assuming you have a ViewModel class
            timer = new System.Timers.Timer(10000); // 10 seconds interval
            timer.Elapsed += Timer_Elapsed;
            timer.Start();
        }

        private async void Timer_Elapsed(object sender, ElapsedEventArgs e)
        {
            try
            {
                // Replace the following with your actual logic for device checking
                var devices = Adb.Devices(DeviceState.Device).ToList();
                var currentDevices = mainVM.Devices.Select(x => x.DeviceId).ToList();
                var newDevices = devices.Except(currentDevices).ToList();
                var offlineDevices = currentDevices.Except(devices).ToList();

                foreach (var online in devices.Intersect(currentDevices))
                    mainVM.Devices.FirstOrDefault(x => x.DeviceId.Equals(online)).IsConnected = true;

                foreach (var offline in offlineDevices.Intersect(currentDevices))
                    mainVM.Devices.FirstOrDefault(x => x.DeviceId.Equals(offline)).IsConnected = false;

                await Dispatcher.InvokeAsync(() =>
                {
                    newDevices.ForEach(x => mainVM.Devices.Add(new DeviceData { DeviceId = x, IsConnected = true }));
                });
            }
            catch (Exception ex)
            {
                if (ex is AggregateException ae)
                    ex = ae.InnerException;

                mainVM.Logs.Add($"{ex.GetType().FullName}: {ex.Message}, {ex.StackTrace}");
            }
            finally
            {
                mainVM.Refresh(); // Assuming you have a Refresh method in your ViewModel
                timer.Start();
            }
        }
    }
}
```

3. **MainViewModel.cs:**
```csharp
using System.Collections.ObjectModel;

namespace YourNamespace
{
    public class MainViewModel
    {
        public ObservableCollection<DeviceData> Devices { get; set; }
        public ObservableCollection<string> Logs { get; set; }

        public MainViewModel()
        {
            Devices = new ObservableCollection<DeviceData>();
            Logs = new ObservableCollection<string>();
        }

        public void Refresh()
        {
            // Implement logic to refresh UI here
        }
    }
}
```

Hãy đảm bảo rằng bạn thay thế `YourNamespace` bằng tên thực tế của không gian tên của dự án bạn và cập nhật các thư viện và logic kiểm tra thiết bị để phản ánh yêu cầu của bạn.
