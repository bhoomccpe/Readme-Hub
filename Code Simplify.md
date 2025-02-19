```C#
        public void StartMonitorDevice(string portname,string sensor, CancellationToken cancellationToken)
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
        // ------- Calibrator -----------------
        public string DetectCalibrator(string calibratorName) 
        {
            List<string> portNames = serialPortService.GetFilteredPortNames();
            _calibrator = CalibratorModelFactory.GetModel(calibratorName);
            foreach (string portName in portNames)
            {
                try
                {
                    // Open each port with the required baud rate
                    serialPortService.OpenConnection(portName, _calibrator.MaxBaudRate);

                    // Send an identification command (e.g., ReadPressureWithUnit)
                    string message = _calibrator.ReadPressureWithUnit();
                    string response = serialPortService.SendData(message, true);

                    // Validate response format for the calibrator
                    if (IsValidCalibratorResponse(response, _calibrator))
                    {
                        Console.WriteLine($"Calibrator detected on port {portName}.");
                        return portName; // Return the detected port name
                    }
                }
                catch (Exception ex)
                {
                    // Handle connection errors or incorrect devices gracefully
                    Console.WriteLine($"Port {portName} is not the calibrator: {ex.Message}");
                }
                finally
                {
                    // Ensure the port is closed after each attempt
                    serialPortService.CloseConnection();
                }
            }
            return string.Empty;
        }
        public async Task<bool> CheckCalibrator(string portName, string calibratorName)
        {
            bool returnValue = false;
            _calibrator = CalibratorModelFactory.GetModel(calibratorName);
            try
            {
                // Run the connection and data operations on a background thread
                await Task.Run(() =>
                {
                    serialPortService.OpenConnection(portName, _calibrator.MaxBaudRate);
                });

                string message = _calibrator.ReadPressureWithUnit();

                // Send data and get response asynchronously
                string response = serialPortService.SendData(message, true);

                if (IsValidCalibratorResponse(response, _calibrator))
                {
                    returnValue = true;
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Port {portName} is not the calibrator: {ex.Message}");
                returnValue = false;
            }
            finally
            {
                // Ensure the connection is closed on a background thread
                await Task.Run(() => serialPortService.CloseConnection());
            }
            return returnValue;
        }
        private bool IsValidCalibratorResponse(string response, ICalibratorModel calibrator)
        {
            try
            {
                // Check for known success codes or expected response format
                if (response.Contains(calibrator.GetReadSuccessCode()))
                {
                    // Further validation of response format (optional)
                    calibrator.GetFormattedPressureResponse(response); // Check if it parses without error
                    return true;
                }
            }
            catch (CalibratorResponseErrorException)
            {
                // The response is not from the expected calibrator
                return false;
            }

            return false;
        }
        public bool IsCalibratorConnected(string calibratorName, string portName) 
        {
            bool retVal = false;
            try 
            {
                _calibrator = CalibratorModelFactory.GetModel(calibratorName);
                serialPortService.OpenConnection(portName, _calibrator.MaxBaudRate);
                string response = serialPortService.SendData(_calibrator.ReadPressureWithUnit(), waitForResponse: true);
                if (response.Contains(_calibrator.GetReadErrorCode()))
                {
                    retVal = false;
                    Console.WriteLine("----- Calibrator Return Error Code ------");
                }
                else
                {
                    retVal = true;
                    Console.WriteLine("----- Calibrator Return Success Code ------");
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.ToString());
                return false;
            }
            finally 
            {
                serialPortService.CloseConnection();
            }
            return retVal; 
        }
        public string SendCommandToCalibrator(string command, string calibratorName, string portName)
        {
            try
            {
                _calibrator = CalibratorModelFactory.GetModel(calibratorName);
                serialPortService.OpenConnection(portName, _calibrator.MaxBaudRate);
                command += _calibrator.GetMessageEnd();
                return "Response From Calibrator: " + serialPortService.SendData(command, waitForResponse: true);
            }
            catch (Exception ex)
            {
                return $"Exception Occur: {ex}";
            }
            finally
            {
                serialPortService.CloseConnection();
            }
        }
```
