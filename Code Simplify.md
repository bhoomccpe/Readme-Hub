```C#
        public void AutomateDevice(string portname,string sensor, CancellationToken cancellationToken)
        {
            Task.Run(async () =>
            {
                try
                {
                    if (_isMonitoringStarted)
                        throw new CetaDeviceException("CETA's monitoring process is running.");

                    CreateDevice(portname);
                    InitiateDevice();
                    SetAutomateParameter(sensor);
                    await Task.Delay(TimeSpan.FromSeconds(2));
                    SetNewCurveAndEvent();
                    StartDevice();
                    _isMonitoringStarted = true;

                    // Block until cancellation is requested
                    cancellationToken.WaitHandle.WaitOne();

                    StopDevice();
                    UnsetDelegateCurve();
                    SetOriginalDeviceMode();
                    Device.Dispose();
                }
                catch (CetaDeviceException e)
                {
                    OnOutputReceivedCETA(e.ToString());
                    Console.WriteLine(e.ToString());
                    throw;
                }
                catch (Exception e)
                {
                    Console.WriteLine(e.ToString());
                    OnOutputReceivedCETA(e.ToString());
                    throw;
                }
                finally
                {
                    _isMonitoringStarted = false;
                    
                }
            });
        }
```
