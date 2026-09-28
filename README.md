# motion-detection-vision-system
# An automated data-processing tracking pipeline utilizing scripting logic to parse real-time data frames and drive analytical insights.



%housekeep
clc;
clear;
close all;

motionThreshold = 30; % sensitivity 
continueProgram = true;

% Initialize webcam with safe fallback handling
try
    cameraObj = webcam();
catch
    fprintf('[NOTICE] Hardware webcam not detected. Switching to simulation mode for testing.\n');
    cameraObj = [];
end

while continueProgram
    totalFrames = 0;
    
    % Get initial reference frame
    if ~isempty(cameraObj)
        firstShot = snapshot(cameraObj);
        refFrame = double(rgb2gray(firstShot));
    else
        refFrame = double(randi([0, 255], 480, 640)); % Simulation frame
    end
    
    fprintf('Detector is armed and looking...\n');
    isMonitoring = true;
    
    while isMonitoring
        totalFrames = totalFrames + 1;
        
        % Capture live frame
        if ~isempty(cameraObj)
            liveShot = snapshot(cameraObj);
            currentGray = double(rgb2gray(liveShot));
        else
            currentGray = refFrame + randn(size(refFrame))*5; % simulate minor changes
        end
        
        imshow(uint8(currentGray));
        title(sprintf('Motion Detection Active | Frames: %d', totalFrames));
        drawnow;
        
        % Calculate movement difference
        diffValue = calculateMotion(refFrame, currentGray);
        
        % Check if movement exceeds limit
        if diffValue > motionThreshold
            triggerAlarm(totalFrames, motionThreshold);
            isMonitoring = false;
        end
    end
    
    continueProgram = askReset();
end

% Get rid of hardware
if ~isempty(cameraObj)
    clear cameraObj;
end
close all;


%% --- Helper Functions ---
function diffValue = calculateMotion(ref, current)
    diffMatrix = abs(ref - current);
    diffValue = mean(diffMatrix(:));
end

function triggerAlarm(totalFrames, motionThreshold)
    fprintf('\n*** ALARM IS OFF ***\n');
    fprintf('Captured %d frames before trigger.\n', totalFrames);
    fprintf('Detection threshold set to: %.2f\n', motionThreshold);
    beep;
end

function choice = askReset()
    answer = input('Reset motion detector? (y/n): ', 's');
    if strcmpi(answer, 'y')
        choice = true;
    else
        choice = false;
        fprintf('Exiting program. Goodbye!\n');
    end
end
