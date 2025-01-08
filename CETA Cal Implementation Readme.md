# Implementation of C# Library (DLL) into CETA Cal Project
The implementation guideline of CETA_Calib DLL to CETA Cal project for enable the automation feature.
## Overview

This guide outlines the process of implementing a C# library (DLL) into the CETA Cal project. Specifically, it details the modifications required in the `CalibrationDataViewModel` class and integration steps for new commands and logic in the calibration workflow.

## Implementation Process

Add **`CETA_Calib`** reference from Automation DLL to **`CETA Cal`** project

### Files to Modify

1. **Class:** `CalibrationDataViewModel`
   **Path:** `CETA_Cal.GUI.Calibration.CalibrationDataViewModel.cs`
2. **View:** `CalibrationDataView`
   **Path:** `CETA_Cal.GUI.Calibration.CalibrationDataView.xaml`

### Steps to Implement

#### 1. Add Automation Controls to `CalibrationDataView.xaml`

Add the "Calibration button" and "Check boxes" under the `ScrollViewer` section in `CalibrationDataView.xaml`:

```xml
<!--#region Automation -->
<Button Content="Calibrate" Command="{Binding OpenCalibratorAutomationViewCommand}" IsEnabled="{Binding IsCalibrateButtonActivate}" HorizontalAlignment="Left" Margin="886,11,0,0" VerticalAlignment="Top" Width="100" RenderTransformOrigin="0.5,0.5" >
    <Button.RenderTransform>
        <TransformGroup>
            <ScaleTransform ScaleX="1"/>
            <SkewTransform/>
            <RotateTransform/>
            <TranslateTransform/>
        </TransformGroup>
    </Button.RenderTransform>
</Button>
<CheckBox IsChecked="{Binding IsPositiveCalibrationChecked, Mode=TwoWay}" Content="Positive" HorizontalAlignment="Left" Margin="885,40,0,0" VerticalAlignment="Top"/>
<CheckBox IsChecked="{Binding IsNegativeCalibrationChecked, Mode=TwoWay}" Content="Negative" HorizontalAlignment="Left" Margin="885,64,0,0" VerticalAlignment="Top"/>
<!--#endregion-->
```

#### 2. Add Automation Region to the `CalibrationDataViewModel` Class

Add the following code under a new `#region Automation` to implement the commands and methods for the calibration automation feature.

```csharp
using CETA_Calib.Model;
using CETA_Cal.GUI.AutomationCalibration;

#region Automation
public ICommand OpenCalibratorAutomationViewCommand { get; }

private void OpenCalibratorAutomationView()
{
    var viewModel = CalibrationAutomationViewModel.Instance;

    foreach (Window window in Application.Current.Windows)
    {
        if (window is CalibrationAutomationView existingView && existingView.DataContext == viewModel)
        {
            MessageBox.Show("Automation Calibration Window is already open.", "Information", MessageBoxButton.OK, MessageBoxImage.Information);
            return;
        }
    }
    CalibrationData data = MapAutomationCalibratorModel(PositiveStep, NegativeStep);
    viewModel.Initialize(data, PositiveStep, NegativeStep);
    CalibrationAutomationView view = new CalibrationAutomationView
    {
        DataContext = viewModel
    };
    view.Show();
    Console.WriteLine(GlobalNormal.ToString());
}

public double CalculatePercentage(double percentage, double total)
{
    return Math.Abs(Math.Round((percentage / 100) * total));
}

private CalibrationData MapAutomationCalibratorModel(ObservableCollection<CalibrationPointModel> PositiveStep, ObservableCollection<CalibrationPointModel> NegativeStep)
{
    Console.WriteLine("NAME: " + GlobalNormal.ToString());
    CalibrationData calibrationData = new CalibrationData
    {
        ModelName = GlobalNormal.CetaName,
        PortName = "-- PLEASE SELECT PORT --",
        ProcessName = ProcessModel.Process,
        Unit = Unit.ToString(),

        PreloadCount = ProcessModel.PreLoadCount,
        PreloadValue = ProcessModel.PreLoadValue,
        DeviceLimit = GlobalNormal.UpRange,

        IsPositivePressureCalibration = IsPositiveCalibrationChecked,
        MaxPressure = IsPositiveCalibrationChecked ? Math.Abs(SetMax) : Math.Abs(SetMin),
        MinPressure = IsPositiveCalibrationChecked ? Math.Abs(MinHigh) : Math.Abs(MaxLow),

        MeasuringTime = ProcessModel.MeasuringTime,
        HoldingTime = ProcessModel.HoldingTime,
        UpwardCycleCount = ProcessModel.UpCount,
        DownwardCycleCount = ProcessModel.DownCount,
        Sensor = Sensor.ToString()
    };

    if (calibrationData.MaxPressure != calibrationData.MinPressure)
    {
        if (IsPositiveCalibrationChecked)
        {
            calibrationData.TargetPressureValues = new double[PositiveStep.Count];
            for (int i = 0; i < PositiveStep.Count; i++)
                calibrationData.TargetPressureValues[i] = PositiveStep[i].Target;
        }
        else
        {
            calibrationData.TargetPressureValues = new double[NegativeStep.Count];
            for (int i = 0; i < NegativeStep.Count; i++)
                calibrationData.TargetPressureValues[i] = NegativeStep[i].Target;
        }
    }
    return calibrationData;
}

private bool _isNegativeCalibrationChecked;
private bool _isPositiveCalibrationChecked;
private bool _isCalibrateButtonActivate;
public bool IsNegativeCalibrationChecked
{
    get { return _isNegativeCalibrationChecked; }
    set
    {
        if (Set(ref _isNegativeCalibrationChecked, value))
        {
            if (value)
            {
                IsPositiveCalibrationChecked = false;
            }
        }
        IsCalibrateButtonActivate = IsPositiveOrNegativeSelect();
    }
}
public bool IsPositiveCalibrationChecked
{
    get { return _isPositiveCalibrationChecked; }
    set
    {
        if (Set(ref _isPositiveCalibrationChecked, value))
        {
            if (value)
            {
                IsNegativeCalibrationChecked = false;
            }
        }
        IsCalibrateButtonActivate = IsPositiveOrNegativeSelect();
    }
}
public bool IsCalibrateButtonActivate
{
    get { return _isCalibrateButtonActivate; }
    set
    {
        Set(ref _isCalibrateButtonActivate, value);
    }
}
private bool IsPositiveOrNegativeSelect()
{
    return IsNegativeCalibrationChecked || IsPositiveCalibrationChecked;
}
#endregion
```

#### 3. Update the Constructor of `CalibrationDataViewModel`

To link the newly added commands with UI buttons, update the class constructor with the following lines:

```csharp
//automation region
OpenCalibratorAutomationViewCommand = new RelayCommand(OpenCalibratorAutomationView);
```

#### 4. Add AutomationCalibration Folder to CETA\_Cal.GUI

Add the `AutomationCalibration` folder to the `CETA_Cal.GUI` directory to organize and manage automation-related components.

### Testing and Validation

1. Rebuild and run the project.

## Conclusion

By following the above steps, the C# library is successfully integrated into the project, enhancing the calibration workflow with additional automation capabilities. Ensure thorough testing to validate functionality and resolve any issues.
