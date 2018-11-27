# cfsd18-cognition-track
[![Build Status](https://travis-ci.org/cfsd/cfsd18-cognition-track.svg?branch=master)](https://travis-ci.org/cfsd/cfsd18-cognition-track)
# cfsd18-cognition-track
[![Build Status](https://travis-ci.org/cfsd/cfsd18-cognition-track.svg?branch=master)](https://travis-ci.org/cfsd/cfsd18-cognition-track)

## How it works

- The function Track:Track and Track:setUp is used to initialize and set up local variables
- The function Track:nextContainer take a reference of a container from cluon:data:Envelope to extract different data according to different message ID
  - If the message ID matches GroundSpeedReading, extract ground speed to local variable m_groundSpeed
  - If the message ID matches GroundSurfaceProperty, extract nSurfacesInframe to local variable m_nSurfacesInframe and also extract surfaceId to local variable m_surfaceId
  - If the message ID matches GroundSurfaceArea, then extract the object ID, surface coordinates for the current surface area
  - Two midpoints will be calculated, the time duration since last message received will also be calculated
  - check if the frame is full, if so then run the surface frame through collectAndRun function
- The function Track:collectAndRun is used to collect surface frame and run it
  - get surface frame information from local variables to remove the negative path points
  - check if it detects a "stop" sign or a "one point" sign
  - using groundspeed value to get preview distance for the vehicle
  - Calculate the whole path length with all collecting path points coordinates
  - compare the path length and preview distance, if the preview distance is longer then the preview distance is set to path length
- Function driveModelSteering
  - The bool variable m_sharp is set to decide which task is executed, the sharp or the steering. and since it is set false in the track.hpp. so it will execute driverModelSteering function, which returns a tuple consists headingRequest, distanceToAimPoint variables, they are the values of azimuthAngle and distance of the aim-point steer in action object. and the steer message will be sent with the m_senderstamp.
- Function driveModelVelocity
  - This function returns a float variable accelerationRequest. when it is positive, it will be assigned to groundAcceleration object and the acceleration message with senderstamp also will be sent through OD4Session. when the accelerationRequest is negative and then the minus accelerationRequest value will be assigned to groundDeceleration and the deceleration message will be sent with sender stamp
  variables involved: groundspeed: normal running speed velocityLimit: the vehicle limit velocity axLimitPositive/axLimitNagative: max vertical acceleration and deaccelaration value ayLimit: lateral velocity limit speedProfile: a vector storing velocity candidates based on expected lateral acceleration curveRadii: a vector storing the return value of curvature-TriCircle function, which is used to calculate the curvature of path for some steps
## OD4Session message in and out 
- receive 
    - opendlv::logic::perception::GroundSurfaceProperty
    - opendlv::logic::perception::GroundSurfaceArea
    - opendlv::proxy::GroundSpeedReading
    - sender: cognition-detectconelane, 211
- send
    - opendlv::system::SignalStatusMessage readySignal
    - opendlv::proxy::GroundAccelerationRequest
    - opendlv::proxy::GroundDecelerationRequest
    - opendlv::logic::action::AimPoint