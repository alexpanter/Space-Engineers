```csharp
// rotor parameters
IMyMotorStator _rotorLeft = null;
IMyMotorStator _rotorRight = null;
float _velocity = 1.75f;
const float _movementAngle = 1.0f;
const float _notchAngle = 10.0f;
bool _isMovingForwards = true;
bool _isRelaxed = true;

// thruster parameters
List<IMyThrust> _atmThrusters;
List<IMyThrust> _ionThrusters;

// thruster lights
List<IMyInteriorLight> _ionThrusterLights;


public void RotorRelax()
{
   // left rotor
   var leftVelocity = _rotorLeft.GetValueFloat("Velocity");
   _rotorLeft.SetValueFloat( (leftVelocity > 0.0f) ? "LowerLimit" : "UpperLimit", 0.0f);
   var leftReverseAction = _rotorLeft.GetActionWithName("Reverse");
   leftReverseAction.Apply(_rotorLeft);

   // right rotor
   var rightVelocity = _rotorRight.GetValueFloat("Velocity");
   _rotorRight.SetValueFloat( (rightVelocity > 0.0f) ? "LowerLimit" : "UpperLimit", 0.0f);
   var rightReverseAction = _rotorRight.GetActionWithName("Reverse");
   rightReverseAction.Apply(_rotorRight);

   _isRelaxed = true;
}


public void RotorMoveBackwardsAbsolute(float angle)
{
   // left rotor
   _rotorLeft.SetValueFloat("UpperLimit", angle);
   _rotorLeft.SetValueFloat("Velocity", _velocity);

   // right rotor
   _rotorRight.SetValueFloat("LowerLimit", -angle);
   _rotorRight.SetValueFloat("Velocity", -_velocity);
}
public void RotorMoveForwardsAbsolute(float angle)
{
   // left rotor
   _rotorLeft.SetValueFloat("LowerLimit", -angle);
   _rotorLeft.SetValueFloat("Velocity", -_velocity);

   // right rotor
   _rotorRight.SetValueFloat("UpperLimit", angle);
   _rotorRight.SetValueFloat("Velocity", _velocity);
}





public void RotorNotch(bool forwards)
{
   // left rotor
   var leftLowerLimit = _rotorLeft.GetValueFloat("LowerLimit");
   var leftUpperLimit = _rotorLeft.GetValueFloat("UpperLimit");
   // moving forwards
   if (forwards && leftLowerLimit > -90.0f)
   {
       if (_isRelaxed) { leftLowerLimit = 0.0f; }
       // checking current motion (given by last command)
       if (_isMovingForwards)
       {
           _rotorLeft.SetValueFloat("LowerLimit", leftLowerLimit - _notchAngle);
       }
       else
       {
           _rotorLeft.SetValueFloat("LowerLimit", leftUpperLimit - _notchAngle);
       }
       _rotorLeft.SetValueFloat("Velocity", -_velocity);
   }
   // moving backwards
   else if(!forwards && leftUpperLimit < 90.0f)
   {
       if (_isRelaxed) { leftUpperLimit = 0.0f; }
       // checking current motion (given by last command)
       if (!_isMovingForwards)
       {
           _rotorLeft.SetValueFloat("UpperLimit", leftUpperLimit + _notchAngle);
       }
       else
       {
           _rotorLeft.SetValueFloat("UpperLimit", leftLowerLimit + _notchAngle);
       }
       _rotorLeft.SetValueFloat("Velocity", _velocity);
   }


   // right rotor
   var rightLowerLimit = _rotorRight.GetValueFloat("LowerLimit");
   var rightUpperLimit = _rotorRight.GetValueFloat("UpperLimit");
   // moving forwards
   if (forwards && rightUpperLimit < 90.0f)
   {
       if (_isRelaxed) { rightUpperLimit = 0.0f; }
       // checking current motion (given by last command)
       if (_isMovingForwards)
       {
           _rotorRight.SetValueFloat("UpperLimit", rightUpperLimit + _notchAngle);
       }
       else
       {
           _rotorRight.SetValueFloat("UpperLimit", rightLowerLimit + _notchAngle);
       }
       _rotorRight.SetValueFloat("Velocity", _velocity);
   }
   // moving backwards
   else if(!forwards && leftUpperLimit > -90.0f)
   {
       if (_isRelaxed) { rightLowerLimit = 0.0f; }
       // checking current motion (given by last command)
       if (!_isMovingForwards)
       {
           _rotorRight.SetValueFloat("LowerLimit", rightLowerLimit - _notchAngle);
       }
       else
       {
           _rotorRight.SetValueFloat("LowerLimit", rightUpperLimit - _notchAngle);
       }
       _rotorRight.SetValueFloat("Velocity", -_velocity);
   }

   _isMovingForwards = forwards;
   _isRelaxed = false;
}





public void IonThrustersSetActivated(bool activate)
{
   String method = (activate) ? "OnOff_On" : "OnOff_Off";

   for(int i = 0; i < _ionThrusters.Count; i++)
   {
       var action = _ionThrusters[i].GetActionWithName(method);
       action.Apply(_ionThrusters[i]);
   }
   for(int i = 0; i < _ionThrusterLights.Count; i++)
   {
       var action = _ionThrusterLights[i].GetActionWithName(method);
       action.Apply(_ionThrusterLights[i]);
   }

   if (!activate) { ThrusterSetZeroOverride(); }
}
public void AtmThrustersSetActivated(bool activate)
{
   String method = (activate) ? "OnOff_On" : "OnOff_Off";

   for(int i = 0; i < _ionThrusters.Count; i++)
   {
       var action = _ionThrusters[i].GetActionWithName(method);
       action.Apply(_ionThrusters[i]);
   }
   for(int i = 0; i < _ionThrusterLights.Count; i++)
   {
       var action = _ionThrusterLights[i].GetActionWithName(method);
       action.Apply(_ionThrusterLights[i]);
   }

   if (!activate) { ThrusterSetZeroOverride(); }
}




public void ThrusterDecreaseOverride()
{
   for(int i = 0; i < _ionThrusters.Count; i++)
   {
        var action = _ionThrusters[i].GetActionWithName("DecreaseOverride");
       action.Apply(_ionThrusters[i]);
   }
}
public void ThrusterIncreaseOverride()
{
   for(int i = 0; i < _ionThrusters.Count; i++)
   {
       var action = _ionThrusters[i].GetActionWithName("IncreaseOverride");
       action.Apply(_ionThrusters[i]);
   }
}
public void ThrusterSetZeroOverride()
{
   for(int i = 0; i < _ionThrusters.Count; i++)
   {
       _ionThrusters[i].SetValueFloat("Override", 0.0f);
   }
}
public void ThrusterSetFullOverride()
{
   float max = 34.6f * 1000.0f * 5f; // maximum override in-game
   for(int i = 0; i < _ionThrusters.Count; i++)
   {
       _ionThrusters[i].SetValueFloat("Override", max);
   }
}


public Program() // constructor, RuntimeInfo.UpdateFrequency to run without timer block (call itself)

{
   // rotor setup
   _rotorLeft = (IMyMotorStator)GridTerminalSystem.GetBlockWithName("AAA_thruster_rotor_left");
   _rotorRight = (IMyMotorStator)GridTerminalSystem.GetBlockWithName("AAA_thruster_rotor_right");
   _rotorLeft.SetValueFloat("Velocity", 0.0f);
   _rotorRight.SetValueFloat("Velocity", 0.0f);


   // thruster setup
   _ionThrusters = new List<IMyThrust>();

   var leftIons = new List<IMyTerminalBlock>();
   GridTerminalSystem.GetBlockGroupWithName("AAA_thrusters_ion_left").GetBlocks(leftIons);
   for(int i = 0; i < leftIons.Count; i++) { if(leftIons[i] is IMyThrust) { _ionThrusters.Add(leftIons[i] as IMyThrust); } }

   var rightIons = new List<IMyTerminalBlock>();
   GridTerminalSystem.GetBlockGroupWithName("AAA_thrusters_ion_right").GetBlocks(rightIons);
   for(int i = 0; i < rightIons.Count; i++) { if(rightIons[i] is IMyThrust) { _ionThrusters.Add(rightIons[i] as IMyThrust); } }

   _atmThrusters = new List<IMyThrust>();

   var leftAtms = new List<IMyTerminalBlock>();
   GridTerminalSystem.GetBlockGroupWithName("AAA_thrusters_atm_left").GetBlocks(leftAtms);
   for(int i = 0; i < leftAtms.Count; i++) { if(leftAtms[i] is IMyThrust) { _atmThrusters.Add(leftAtms[i] as IMyThrust); } }

   var rightAtms = new List<IMyTerminalBlock>();
   GridTerminalSystem.GetBlockGroupWithName("AAA_thrusters_atm_right").GetBlocks(rightAtms);
   for(int i = 0; i < rightAtms.Count; i++) { if(rightAtms[i] is IMyThrust) { _atmThrusters.Add(rightAtms[i] as IMyThrust); } }


   // thruster lights
   _ionThrusterLights = new List<IMyInteriorLight>();
   var ionThrusterLightsTemp = new List<IMyTerminalBlock>();
   GridTerminalSystem.GetBlockGroupWithName("AAA_thrusters_ion_lights").GetBlocks(ionThrusterLightsTemp);
   for (int i = 0; i < ionThrusterLightsTemp.Count; i++)
   {
       if (ionThrusterLightsTemp[i] is IMyInteriorLight) { _ionThrusterLights.Add(ionThrusterLightsTemp[i] as IMyInteriorLight); }
   }

}



public void Save() // when program needs to save its state

{
   RotorRelax();

}



public void Main(string argument, UpdateType updateSource)

{
   switch(argument)
   {
   // rotor adjustment
   case "relax": RotorRelax(); break;
   case "forwards": RotorNotch(forwards: true); break;
   case "backwards": RotorNotch(forwards: false); break;
   case "full_forwards": RotorMoveForwardsAbsolute(90.0f); break;
   case "full_backwards": RotorMoveBackwardsAbsolute(90.0f); break;

   // thrust ignition
   case "ion_thrusters_on": IonThrustersSetActivated(activate: true); break;
   case "ion_thrusters_off": IonThrustersSetActivated(activate: false); break;
   case "atm_thrusters_on": AtmThrustersSetActivated(activate: true); break;
   case "atm_thrusters_off": AtmThrustersSetActivated(activate: false); break;

   // thrust override
   case "zero_thrust": ThrusterSetZeroOverride(); break;
   case "full_thrust": ThrusterSetFullOverride(); break;
   case "increase_override": ThrusterIncreaseOverride(); break;
   case "decrease_override": ThrusterDecreaseOverride(); break;

   default:
       break;
   }

}
```
