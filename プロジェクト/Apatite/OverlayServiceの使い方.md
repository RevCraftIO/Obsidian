
## ViewModel
```CSharp
    [RelayCommand]
    private async Task ShowProgressWithValue()
    {
        await overlayService.ShowProgressOverlayWithValueAsync("データ処理中...", async (progress) =>
        {
            for (int i = 0; i <= 100; i += 5)
            {
                await Task.Delay(500);
                progress.Report(i);
            }
        });
    }

    [RelayCommand]
    private async Task ShowProgress()
    {
        await overlayService.ShowProgressOverlayAsync("データ処理中...", async() =>
        {
            for (int i = 0; i <= 100; i += 5)
            {
                await Task.Delay(500);
            }
        });
    }
```

## View
```XML

    <Grid>
        <StackPanel
            VerticalAlignment="Center"
            HorizontalAlignment="Center">
            <Button
                Content="ShowProgress"
                VerticalAlignment="Center"
                HorizontalAlignment="Center"
                Margin="0,0,0,15"
                Command="{Binding ViewModel.ShowProgressCommand}"/>
            <Button
                Content="ShowProgressWithValue"
                VerticalAlignment="Center"
                HorizontalAlignment="Center"
                Command="{Binding ViewModel.ShowProgressWithValueCommand}"/>
        </StackPanel>
    </Grid>
```
