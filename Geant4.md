## reset global time

对于放射性衰变, 通常其半衰期长达几年甚至更久, 这种时候global time的精度可能达不到要求. 也即后续的track的global time都是在衰变的时间基础上累加一个ns甚至是ps量级的时间, 这种时候这种后续的时间是无法进行区分的. 在这种情况下, 需要对衰变产生的次级track进行操作, 调整其global time. 在Geant4中, 每一个step中产生的次级track默认存储在`furgentstack`里面, 通过`stackingAction`可以进行操作. 此外在`steppingaction`中也提供了获得次级track的接口.

1. 通过`aStep->GetSecondaryInCurrentStep();`可以得到当前`step`中产生的次级`track`常指针构成的`vecotr`. 遍历该`vector`重置globaltime即可. 需要注意的是这里存储的都是常指针, 为了修改其中的成员, 需要用到**强制类型转换`const_cast`去除常量属性**. 具体程序如下:

```cpp
void SteppingAction::UserSteppingAction(const G4Step* aStep)
{
    if ( aStep->GetTrack()->GetParticleDefinition()->GetParticleName() == "Co60") {
        if (aStep->GetPostStepPoint()->GetProcessDefinedStep()
            							->GetProcessName()=="RadioactiveDecay"){
            const std::vector<const G4Track*>* secondaries = 
              							aStep->GetSecondaryInCurrentStep();
            for (auto atrack : *secondaries){
                // G4double pretime = aStep->GetPreStepPoint()->GetGlobalTime();
                // (const_cast<G4Track *>(atrack))->SetGlobalTime(pretime);
                (const_cast<G4Track *>(atrack))->SetGlobalTime(0.);
                G4cout << "Global time changed.\n ";
            }
        }
    }
}
```

2. 在模拟过程中每次产生的track其实都存储在`stack`里面, 详见后续中`G4StackingAction`的介绍. 程序如下: 其中`StackManager`中的`ReClassify()`函数会调用`StackingAction`中的`G4ClassificationOfNewTrack StackingAction::ClassifyNewTrack(const G4Track * aTrack)`, 该函数默认在每次新track进栈的时候自动调用. 在这里可以对进栈之前的track进行操作. 通过在`steppingAction`中确认`trigger`的触发, 在`StackingAction`根据`trigger`对进栈的`track`进行操作. 需要注意的是, `ReClassify()`调用的`ClassifyNewTrack()`并不是在调用的时候生效的, 而是在当前`step`结束的时候统一进栈的. 所以在`SteppingAciton`中将`trigger`重置为`false`是无效的. 个人认为在`trackAciton`的结束重置`trigger`是有效的(未经验证). 这里除了`trigger`之外, 加入了`trackid`的另一重判断, 也即只有`trigger`触发时, 且进栈的track的`parentid`是当前的`trackid`时才进行重置的操作. 同样的, 这里一样需要`const_cast`去除常属性.

```cpp
void SteppingAction::UserSteppingAction(const G4Step* aStep)
{
    if ( aStep->GetTrack()->GetParticleDefinition()->GetParticleName() == "Co60") {
        if (aStep->GetPostStepPoint()->GetProcessDefinedStep()->GetProcessType()==6){ 
            fStack->SetTriggerAndID(true, aStep->GetTrack()->GetTrackID());
            fStack->SetBaseTime(0.);
            G4EventManager::GetEventManager()->GetStackManager()->ReClassify();
        }
    }
}

G4ClassificationOfNewTrack
StackingAction::ClassifyNewTrack(const G4Track * aTrack)
{
    if ( fTrigger && (aTrack->GetParentID() == curid ) ){
        G4cout << "calling the method of ClassifyNewTrack" << G4endl;
        (const_cast<G4Track *>(aTrack))->SetGlobalTime(basetime);
    }
    return fUrgent;
}
```



## G4VProcess

One can get the process definition via `G4VProcess`. [ProcessSubType of EM](http://geant4.web.cern.ch/geant4/collaboration/working_groups/electromagnetic/)


```cpp
const G4String& G4VProcess::GetProcessTypeName(G4ProcessType aType ) 
{
  switch (aType) {
  case fNotDefined:         return typeNotDefined; break;
  case fTransportation:     return typeTransportation; break;
  case fElectromagnetic:    return typeElectromagnetic; break;
  case fOptical:            return typeOptical; break;
  case fHadronic:           return typeHadronic; break;
  case fPhotolepton_hadron: return typePhotolepton_hadron; break;
  case fDecay:              return typeDecay; break;
  case fGeneral:            return typeGeneral; break;
  case fParameterisation:   return typeParameterisation; break;
  case fUserDefined:        return typeUserDefined; break;
  case fPhonon:             return typePhonon; break;
  default: ;
  }

  return noType;  
}

 G4int G4VProcess::GetProcessSubType() const
{
  return theProcessSubType;
}
```



```cpp
G4VProcess * process = aStep->GetPostStepPoint()->GetProcessDefinedStep();
process->GetProcessSubType();
```

To find out the name of process sub type, try searching "SetProcessSubType" in Geant4 source finder.

## Particle Codes

The [Particle Data Group particle code](http://home.fnal.gov/~mrenna/lutp0613man2/node44.html) is used consistently throughout the program.



## G4UImessenger

In `PhysicsList::SetCuts()`, the method of `DumpCutValuesTable()` will be invoked  `if (verboseLevel>0) DumpCutValuesTable();`. Here, the `verboseLevel` is defined in the base class `G4VUserPhysicsList`, and can be change via UI command `/run/particle/verbose`.



For more details about `G4UImessenger`, please refer to [section 7.2.2](http://geant4.cern.ch/UserDocumentation/UsersGuides/ForApplicationDeveloper/html/ch07s02.html) of Geant4 application user's guide.

```cpp
void G4UIcommand::availableForStates(G4ApplicationState s1,...);
      // These methods define the states where the command is available.
      // Once one of these commands is invoked, the command application will
      // be denied when Geant4 is NOT in the assigned states. 
```

If your command is valid only for certain states of the Geant4 kernel, specify these states by this method. Currently available states are `G4State_PreInit, G4State_Init, G4State_Idle, G4State_GeomClosed,` and `G4State_EventProc`. Refer to the [section 3.4.2](http://geant4.cern.ch/UserDocumentation/UsersGuides/ForApplicationDeveloper/html/ch03s04.html) for meaning of each state. Please note that the `Pause` state had been removed from `G4ApplicationState`.

`G4State_PreInit` state: A Geant4 application starts with this state. The application needs to be initialized when it is in this state. The application occasionally comes back to this state if geometry, physics processes, and/or cut-off have been changed after processing a run.

`G4State_Init` state: The application is in this state while the `Initialize()` method of `G4RunManager` is being invoked. Methods defined in any *user initialization* classes are invoked during this state.

`G4State_Idle` state: The application is ready for starting a run.

`G4State_GeomClosed` state: When `BeamOn()` is invoked, the application proceeds to this state to process a run. Geometry, physics processes, and cut-off cannot be changed during run processing.

`G4State_EventProc` state: A Geant4 application is in this state when a particular event is being processed. `GetCurrentEvent()` and `GetPreviousEvent()` methods of `G4RunManager` are available only at this state.

`G4State_Quit` state: When the destructor of `G4RunManager` is invoked, the application comes to this "dead end" state. Managers of the Geant4 kernel are being deleted and thus the application cannot come back to any other state.

`G4State_Abort` state: When a `G4Exception` occurs, the application comes to this "dead end" state and causes a core dump. The user still has a hook to do some "safe" opperations, e.g. storing histograms, by implementing a user concrete class of `G4VStateDependent`. The user also has a choice to suppress the occurence of `G4Exception` by a UI command */control/suppressAbortion*. When abortion is suppressed, you will still get error messages issued by G4Exception, and there is NO guarantee of a correct result after the G4Exception error message.



```cpp
inline void G4UIcommand::SetToBeBroadcasted(G4bool val);
      // If toBeBroadcasted is set to false, this command won't be sent to worker threads.
      // This toBeBroadcasted parameter could be changed with SetToBeBroadcasted() method
      // except for G4UIdirectory.
```

**Attention: If multi-threading is supposed to be applied, be careful about this method.**



```cpp
// G4UIcmdWithADoubleAndUnit.hh
void SetParameterName(const char * theName,G4bool omittable,
                          G4bool currentAsDefault=false);
    //  Set the parameter name for double parameterxs. Name is used by
    // the range checking routine.
    //  If "omittable" is set as true, the user of this command can ommit
    // the value when he/she applies the command. If "omittable" is false,
    // the user must supply a value.
    //  "currentAsDefault" flag is valid only if "omittable" is true. If this
    // flag is true, the current value is used as the default value when the
    // user ommit the double parameter. If this flag is false, the value
    // given by the next SetDefaultValue() method is used.
```

`name` is used for the range checking, which can be defined via `SetRange(const char* rs)` method. `omittable` (可省略的) defined whether the value is mandatory or optional. 



```cpp
  void SetUnitCategory(const char * unitCategory);
  void SetUnitCandidates(const char * candidateList);
  void SetDefaultUnit(const char * defUnit);
  //  These three methods must be used alternatively.
  //  The user cannot ommit the unit as the second parameter of the command if
  // SetUnitCategory() or SetUnitCandidates() is used, while the unit defined
  // by SetDefaultUnit() method is used as the default unit so that the user can
  // ommits the second parameter.
  //  SetUnitCategory() defines the category of the units which will be accepted.
  // The available categories can be found in G4SystemOfUnits.hh in global category.
  // Only the units categorized in the given category are accepted as the second
  // parameter of the command.
  //  SetUnitCandidates() defines the candidates of units. Units listed in the
  // argument of this method must be separated by space(s). Only the units listed
  // in the candidate list are accepted as the second parameter of the command.
  //  SetDefaultUnit() defines the default unit and also it defines the category
  // of the allowed units. Thus only the units categorized as the given default
  // unit will be accepted.
```

`SetDefaultUnit()` defines both the default unit and the category (just like `SetUnitCategory`). With the invocation of `SetDefaultUnit()`, there is no need to input the second parameter. Thus, the method of `SetDefaultUnit()` is suggested.



## Multi thread

**根据测试结果显示, 如果存储每个step的信息, 在多线程的情况下, 使用`"g4root.hh"`作为analysis manager, 并进行merge`analysisManager->SetNtupleMerging(true);`, 从结果上看, 每个step的结果其实是混乱的, 如果不进行merge: `analysisManager->SetNtupleMerging(true, nofThreads);`貌似可以得到正确的结果, 只是结果都单独存在不同的root file中**

### [Analysis Manager Classes](https://geant4.web.cern.ch/geant4/UserDocumentation/UsersGuides/ForApplicationDeveloper/html/ch09s02.html)

Seeing section 9.2.7 of UsersGuidesForApplicationDeveloper.

Since Geant4 version 10.3 it is possible to activate merging of ntuples with ROOT output type:

```cpp
auto analysisManager = G4AnalysisManager::Instance();
analysisManager->SetNtupleMerging(true);
```

The ntuples produced on workers will be then progressively being merged to the main ntuples on the master. By default, the ntuples are written at the same file as the final histograms. Users can also select merging in a given number of files or change the default basket size value (32000) :

```cpp
auto analysisManager = G4AnalysisManager::Instance();
G4int nofReducedNtupleFiles = 2;
G4int basketSize = 64000;
analysisManager->SetNtupleMerging(true, nofReducedNtupleFiles, basketSize);
```

No merging of ntuples is provided with CSV and AIDA XML formats.

这里`void SetNtupleMerging(G4bool mergeNtuples, G4int nofReducedNtupleFiles = 0, unsigned int basketSize = fgkDefaultBasketSize);`控制Tree的合并(merging), 其中`mergeNtuples`最好是`true`, 如果设置为`analysisManager->SetNtupleMerging(false);`, 最终好像会将Tree文件视为empty并删除. 原因在于: 在false的情况下, `fNtupleMergeMode`会被设置为`G4NtupleMergeMode::kNone`. 且不知为何`fNtupleManager->IsEmpty()==true`, 暂时尚不清楚具体原因.

```cpp
// G4RootAnalysisManager.cc
  // Delete files if empty in MT mode 
  if ( ( fState.GetIsMaster() && 
         fH1Manager->IsEmpty() && fH2Manager->IsEmpty() && fH3Manager->IsEmpty() &&
         fP1Manager->IsEmpty() && fP2Manager->IsEmpty() && fNtupleManager->IsEmpty() ) || 
       ( ( ! fState.GetIsMaster() ) && fNtupleManager->IsEmpty() &&
             fNtupleMergeMode == G4NtupleMergeMode::kNone ) ) {
```

如果不想合并文件, 可以通过`analysisManager->SetNtupleMerging(true, nofThreads);` 这里`nofReducedNtupleFiles`为合并后的Tree文件数, 设置为`nofThreads`总线程数, 最终会生成等于总线程数的文件(也即等效于不合并文件).





## Optical process (Get boundary and step status)

Optical photon will be processed at the boundary. The boundary process status is used to describe this.

```cpp
// defined in G4OpBoundaryProcess.hh
enum G4OpBoundaryProcessStatus {  Undefined,
                                  Transmission, FresnelRefraction,
                                  FresnelReflection, TotalInternalReflection,
                                  LambertianReflection, LobeReflection,
                                  SpikeReflection, BackScattering,
                                  Absorption, Detection, NotAtBoundary,
                                  SameMaterial, StepTooSmall, NoRINDEX,
                                  PolishedLumirrorAirReflection,
                                  PolishedLumirrorGlueReflection,
                                  PolishedAirReflection,
                                  PolishedTeflonAirReflection,
                                  PolishedTiOAirReflection,
                                  PolishedTyvekAirReflection,
                                  PolishedVM2000AirReflection,
                                  PolishedVM2000GlueReflection,
                                  EtchedLumirrorAirReflection,
                                  EtchedLumirrorGlueReflection,
                                  EtchedAirReflection,
                                  EtchedTeflonAirReflection,
                                  EtchedTiOAirReflection,
                                  EtchedTyvekAirReflection,
                                  EtchedVM2000AirReflection,
                                  EtchedVM2000GlueReflection,
                                  GroundLumirrorAirReflection,
                                  GroundLumirrorGlueReflection,
                                  GroundAirReflection,
                                  GroundTeflonAirReflection,
                                  GroundTiOAirReflection,
                                  GroundTyvekAirReflection,
                                  GroundVM2000AirReflection,
                                  GroundVM2000GlueReflection,
                                  Dichroic };
```

Besides, the `G4StepStatus` can be used to describe whether the step point is in the boundary. For example, if the step status is `fGeomBoundary`, which means the particle reaches to the boundary.

```cpp
//////////////////
enum G4StepStatus
//////////////////
{
  fWorldBoundary,           
    // Step reached the world boundary
  fGeomBoundary,            
    // Step defined by a geometry boundary
  fAtRestDoItProc,          
    // Step defined by a PreStepDoItVector
  fAlongStepDoItProc,       
    // Step defined by a AlongStepDoItVector
  fPostStepDoItProc,        
    // Step defined by a PostStepDoItVector
  fUserDefinedLimit,
    // Step defined by the user Step limit in the logical volume
  fExclusivelyForcedProc,   
    // Step defined by an exclusively forced PostStepDoIt process 
  fUndefined                
    // Step not defined yet
};
```

One can get the information of `G4StepStatus` and `G4OpBoundaryProcessStatus` in `SteppingAction`.

```cpp
// the method to get step status
G4StepStatus stepStatus = fUndefined; // step status
if (aStep->GetPostStepPoint())
    stepStatus = aStep->GetPostStepPoint()->GetStepStatus();
```

Two methods can be used to get boundary process status of an optical photon.

The two methods are similar. The basic idea is try to get the current optical boundary process which is registered via `PhysicsList`. To get the process of optical photon, one can use `aStep->GetTrack()->GetDefinition()->GetProcessManager()` or `G4OpticalPhoton::OpticalPhoton()->GetProcessManager();`. Since in this track, the current particle is optical photon, which means `aStep->GetTrack()->GetDefinition()` return an optical photon. 



```cpp
// first method
G4OpBoundaryProcessStatus boundaryStatus = Undefined; // boundary status
G4OpBoundaryProcess* boundary = NULL; // boundary process

G4ProcessManager* pm    //get process manager of current particle
    = aStep->GetTrack()->GetDefinition()->GetProcessManager();

G4int nprocesses = pm->GetProcessListLength(); // get number of registered process for current particle(optical photon)
G4ProcessVector* pv = pm->GetProcessList();	// get vector of registered process for current particle(optical photon)

for( G4int i=0;i<nprocesses;i++){
    G4cout << "(*pv) is " << (*pv)[i]->GetProcessName() << G4endl;
    if((*pv)[i]->GetProcessName()=="OpBoundary"){
        boundary = (G4OpBoundaryProcess*)(*pv)[i]; // get current boundary process
        break;
    }
}
boundaryStatus = boundary->GetStatus();
```

```cpp
// second method
G4OpBoundaryProcessStatus theStatus = Undefined;

G4ProcessManager* OpManager =
                    G4OpticalPhoton::OpticalPhoton()->GetProcessManager();

if (OpManager) {
    G4int MAXofPostStepLoops =
            OpManager->GetPostStepProcessVector()->entries(); // Get number of post step process (actually is the number of
                                                              // registered process for optical photon)
    G4ProcessVector* fPostStepDoItVector =
            OpManager->GetPostStepProcessVector(typeDoIt);

    for ( G4int i=0; i<MAXofPostStepLoops; i++) {
        G4VProcess* fCurrentProcess = (*fPostStepDoItVector)[i];
        G4cout << "fCurrentProcess is " << fCurrentProcess->GetProcessName()<<G4endl;
        G4OpBoundaryProcess* fOpProcess = dynamic_cast<G4OpBoundaryProcess*>(fCurrentProcess);// get current boundary process
        if (fOpProcess) { theStatus = fOpProcess->GetStatus(); break;}
    }
}
```



## Rotation

### Rotation of G4SPSPosDistribution

In the method of `G4SPSPosDistribution`, one can generate raw position according to the previously defined source position type and shape.

```cpp
void G4SPSPosDistribution::SetPosRot1(G4ThreeVector posrot1)
{
  // This should be x'
  Rotx = posrot1;
  if(verbosityLevel == 2)
    {
      G4cout << "Vector x' " << Rotx << G4endl;
    }
  GenerateRotationMatrices();
}

void G4SPSPosDistribution::SetPosRot2(G4ThreeVector posrot2)
{
  // This is a vector in the plane x'y' but need not
  // be y'
  Roty = posrot2;
  if(verbosityLevel == 2)
    {
      G4cout << "The vector in the x'-y' plane " << Roty << G4endl;
    }
  GenerateRotationMatrices();
}

void G4SPSPosDistribution::GenerateRotationMatrices()
{
  // This takes in 2 vectors, x' and one in the plane x'-y',
  // and from these takes a cross product to calculate z'.
  // Then a cross product is taken between x' and z' to give
  // y'.
  Rotx = Rotx.unit(); // x'
  Roty = Roty.unit(); // vector in x'y' plane
  Rotz = Rotx.cross(Roty); // z'
  Rotz = Rotz.unit();
  Roty =Rotz.cross(Rotx); // y'
  Roty =Roty.unit();
  if(verbosityLevel == 2)
    {
      G4cout << "The new axes, x', y', z' " << Rotx << " " << Roty << " " << Rotz << G4endl;
    }
}
```

The above three methods are used to define new coordinate system. With the method of `SetPosRot1`, the vector $$x^\prime$$ in the current coordinate system should be defined. With the method of `SetPosRot2`, the arbitrary vector in the plane of $$x^\prime-y^\prime$$ except for the vector $$x^\prime$$ in the current coordinate system should be defined. Here, the two vectors don't need to be unit vectors. The following equations demonstrate the conversion of coordinate.

$$\vec { i^{\prime}  } =Rotx\cdot \left[ \begin{matrix} \vec { i }  \\ \vec { j }  \\ \vec { k }  \end{matrix} \right] ,\ \vec { j^{\prime}  } =Roty\cdot \left[ \begin{matrix} \vec { i }  \\ \vec { j }  \\ \vec { k }  \end{matrix} \right],\ \vec { k^{\prime}  } =Rotz\cdot \left[ \begin{matrix} \vec { i }  \\ \vec { j }  \\ \vec { k }  \end{matrix} \right]$$

$$convert\ matrix:\ A=\left[ \begin{matrix} Rotx \\ Roty \\ Rotz \end{matrix} \right]=\left[ \begin{matrix} x_x &y_x  &z_x  \\  x_y&  y_y& z_y \\ x_z &y_z  &z_z  \end{matrix} \right] ,\ \left[ \begin{matrix} \vec { i^\prime }  \\ \vec { j\prime }  \\ \vec { k\prime }  \end{matrix} \right]=A\cdot\left[ \begin{matrix} \vec { i }  \\ \vec { j }  \\ \vec { k }  \end{matrix} \right]$$

$$\left[ \begin{matrix} x & y & z \end{matrix} \right] \cdot \left[ \begin{matrix} \vec { i }  \\ \vec { j }  \\ \vec { k }  \end{matrix} \right] =\left[ \begin{matrix} x ^\prime& y^\prime & z^\prime \end{matrix} \right] \cdot \left[ \begin{matrix} \vec { i^\prime }  \\ \vec { j^\prime }  \\ \vec { k^\prime }  \end{matrix} \right] =\left[ \begin{matrix} x ^\prime& y^\prime & z^\prime \end{matrix} \right] \cdot A\cdot \left[ \begin{matrix} \vec { i }  \\ \vec { j }  \\ \vec { k }  \end{matrix} \right] $$

$$so,\ \left[ \begin{matrix} x & y & z \end{matrix} \right]=\left[ \begin{matrix} x ^\prime& y^\prime & z^\prime \end{matrix} \right] \cdot A$$

$$i.e.\ x=x^\prime \cdot x_x + y^\prime \cdot x_y + z^\prime \cdot x_z, \ y=x^\prime \cdot y_x + y^\prime \cdot y_y + z^\prime \cdot y_z,\ z=x^\prime \cdot z_x + y^\prime \cdot z_y + z^\prime \cdot z_z$$

在程序中也验证了上述过程, 在`G4SPSPosDistribution`抽样产生位置后, 加入了下述处理过程:

```cpp
RandPos.setX(x);
RandPos.setY(y);
RandPos.setZ(z);

// Apply Rotation Matrix
// x * Rotx, y * Roty and z * Rotz
tempx = (x * Rotx.x()) + (y * Roty.x()) + (z * Rotz.x());
tempy = (x * Rotx.y()) + (y * Roty.y()) + (z * Rotz.y());
tempz = (x * Rotx.z()) + (y * Roty.z()) + (z * Rotz.z());

RandPos.setX(tempx);
RandPos.setY(tempy);
RandPos.setZ(tempz);
```

在抽样产生位置时, 首先, 通过$$Rotx\ Roty \ Rotz$$确定$$\vec{i^\prime}\ \vec{j^\prime}\ \vec{k^\prime}$$, 这里$$\vec{i^\prime}\ \vec{j^\prime}\ \vec{k^\prime}$$, 实际上抽样产生在$$\vec{i^\prime}\ \vec{j^\prime}\ \vec{k^\prime}$$坐标系下的位置坐标$$(x^\prime,\ y^\prime,\ z^\prime)$$, 再将该坐标通过转化矩阵$$A$$转换为$$\vec{i}\ \vec{j}\ \vec{k}$$坐标系下的位置坐标$$(x,\ y,\ z)$$.

**在抽样产生位置时, 默认是按照沿z轴方向的几何进行抽样, 比如对于一个旋转后的圆柱体, 默认抽样产生的是沿z轴布置的中心在$$CentreCoords$$的圆柱体, 也即在$$\vec{i^\prime}\ \vec{j^\prime}\ \vec{k^\prime}$$坐标系下的位置坐标$$(x^\prime,\ y^\prime,\ z^\prime)$$. 基于此, 这里可以确定$$\vec{i^\prime}\ \vec{j^\prime}\ \vec{k^\prime}$$即为默认圆柱体的三维坐标系. 实际的坐标系即为当前的坐标系, 在该坐标系下圆柱体并不是沿z轴放置的. 因此, 在设定$$Rotx\ Roty \ Rotz$$确定$$\vec{i^\prime}\ \vec{j^\prime}\ \vec{k^\prime}$$, 只需在圆柱体中心横截面上选择两个$$vector$$即可.**



## G4SPSDistribution

These methods are used to generate the position/direction/energy of a primary vertex according to the defined distribution with three similar classes: `G4SPSPosDistribution`, `G4SPSAngDistribution`, `G4SPSEneDistribution`. **Before generating direction using `G4SPSAngDistribution`, the `G4SPSPosDistribution` should be defined.**

```cpp
biasRndm = new G4SPSRandomGenerator();
posGenerator = new G4SPSPosDistribution();
posGenerator->SetBiasRndm(biasRndm);
angGenerator = new G4SPSAngDistribution();
angGenerator->SetPosDistribution(posGenerator);// **mandatory 
angGenerator->SetBiasRndm(biasRndm);
eneGenerator = new G4SPSEneDistribution();
eneGenerator->SetBiasRndm(biasRndm);
verbosityLevel = vL;
posGenerator->SetVerbosity(vL);
angGenerator->SetVerbosity(vL);
eneGenerator->SetVerbosity(vL);

// Position stuff
particle_position = posGenerator->GenerateOne();
// Angular stuff
particle_momentum_direction = angGenerator->GenerateOne();
// Energy stuff
particle_energy = eneGenerator->GenerateOne(particle_definition);
```



## Primary Generation

The detailed information can be found in the section 2.6 and 2.7 of Geant4 manual. The examples are can be seen in the [eventgenerator/particleGun](/home/pxy/Software/MJSW/geant4/geant4.10.03.p01/examples/extended/eventgenerator/particleGun).

Two classes can be used: `G4ParticleGun` and `G4GeneralParticleSource`.

### G4ParticleGun

The following methods are provided by `G4ParticleGun`, and all of them can be invoked from the generatePrimaries() method in your concrete G4VUserPrimaryGeneratorAction class.

* void SetParticleDefinition(G4ParticleDefinition*)
* void SetParticleMomentum(G4ParticleMomentum)
* void SetParticleMomentumDirection(G4ThreeVector)
* void SetParticleEnergy(G4double)
* void SetParticleTime(G4double)
* void SetParticlePosition(G4ThreeVector)
* void SetParticlePolarization(G4ThreeVector)
* void SetNumberOfParticles(G4int)

The basic usage of `G4ParticleGun`:

```cpp
G4int n_particle = 1;
fParticleGun  = new G4ParticleGun(n_particle);

G4ParticleDefinition* particle
           = G4ParticleTable::GetParticleTable()->FindParticle("geantino");
fParticleGun->SetParticleDefinition(particle);

fParticleGun->SetParticlePosition(G4ThreeVector(x0,y0,z0));
fParticleGun->SetParticleMomentumDirection(G4ThreeVector(ux,uy,uz));
fParticleGun->SetParticleEnergy(1.0*MeV);
fParticleGun->SetParticleTime(1.*ns);// optional but suggested

fParticleGun->GeneratePrimaryVertex(anEvent);
```

With the method of `GeneratePrimaryVertex(G4Event* evt)`, one can generate the particles in certain vertex. The method is as following:

```cpp
void G4ParticleGun::GeneratePrimaryVertex(G4Event* evt)
{
  if(!particle_definition) 
  {
    G4ExceptionDescription ED;
    ED << "Particle definition is not defined." << G4endl;
    ED << "G4ParticleGun::SetParticleDefinition() has to be invoked beforehand." << G4endl;
    G4Exception("G4ParticleGun::GeneratePrimaryVertex()","Event0109",
    FatalException, ED);
    return;
  }
  // create a new vertex
  G4PrimaryVertex* vertex = 
    new G4PrimaryVertex(particle_position,particle_time);

  // create new primaries and set them to the vertex
  G4double mass =  particle_definition->GetPDGMass();
  for( G4int i=0; i<NumberOfParticlesToBeGenerated; i++ ){
    G4PrimaryParticle* particle =
      new G4PrimaryParticle(particle_definition);
    particle->SetKineticEnergy( particle_energy );
    particle->SetMass( mass );
    particle->SetMomentumDirection( particle_momentum_direction );
    particle->SetCharge( particle_charge );
    particle->SetPolarization(particle_polarization.x(),
                  particle_polarization.y(),
                  particle_polarization.z());
    vertex->SetPrimary( particle );
  }

  evt->AddPrimaryVertex( vertex );
}
```

Where `NumberOfParticlesToBeGenerated` can be defined with `G4ParticleGun::G4ParticleGun(G4int numberofparticles)` or `void SetNumberOfParticles(G4int i)`. With the `for loop` inside `G4ParticleGun`, we can see that if 

Besides, we can see that the method of `GeneratePrimaryVertex` is mainly consist with two individual methods: `G4PrimaryParticle` and `G4PrimaryVertex`. 

Where `G4PrimaryVertex` is used to define the vertex (i.e. the starting position) of particle. The invocation of `G4PrimaryVertex` including three methods. The main parameters of this class is position and time, which can be further changed with `void SetPosition(G4double x0,G4double y0,G4double z0)` and `void SetT0(G4double t0)`.

```cpp
G4PrimaryVertex();
G4PrimaryVertex(G4double x0,G4double y0,G4double z0,G4double t0);
G4PrimaryVertex(G4ThreeVector xyz0,G4double t0);
```

`G4PrimaryParticle` is used to define the particle properties including particle type, particle position, particle energy, particle direction and etc.

The vertex and primary will be combined as the real primary particle using `void G4PrimaryVertex::SetPrimary(G4PrimaryParticle * pp)`:

```cpp
inline void G4PrimaryVertex::SetPrimary(G4PrimaryParticle * pp)
{ 
  if(theParticle == 0) { theParticle = pp;     }
  else                 { theTail->SetNext(pp); }
  theTail = pp;
  numberOfParticle++;
}
```

It can be seen that the method can be invoked more than one time, which means several particles can be generated in same position.

The macro  command directory of the class is `/gun`.

### G4GeneralParticleSource

The macro  command directory of the class is `/gps`, seeing section 2.7 of Geant4 manual. `G4GeneralParticleSource` is used exactly the same way as `G4ParticleGun` in a Geant4 application, and may be substituted for the latter by "global search and replace" in existing application source code.

With the command of "/gps/pos/confine Crystal", one can generate primaries in confined volume. Here the name of "Crystal" is the name of physical volume. **Note, the invocation of gps macro command should be applied after "/run/initialize" because until then the geometry is constructed.**

The macro command of this class in very convenient, [example](/home/pxy/Software/MJSW/geant4/geant4.10.03.p01/examples/extended/eventgenerator/GunGPS)..

### some ideas of generation multiple particles

**First method**: Generating several vertices and particles per event ([example](/home/pxy/Software/MJSW/geant4/geant4.10.03.p01/examples/extended/eventgenerator/particleGun/src/PrimaryGeneratorAction1.cc)):

```cpp
//vertex 1 uniform on cylinder
//
fParticleGun->SetParticlePosition(G4ThreeVector(x1,y1,z1));
fParticleGun->SetParticleMomentumDirection(G4ThreeVector(ux1,uy1,uz2));    
fParticleGun->SetParticleEnergy(1*MeV);
fParticleGun->GeneratePrimaryVertex(anEvent);

//particle 2 at vertex 2
//   
fParticleGun->SetParticlePosition(G4ThreeVector(x2,y2,z2));
fParticleGun->SetParticleMomentumDirection(G4ThreeVector(ux2,uy2,uz2));    
fParticleGun->SetParticleEnergy(1*keV);
fParticleGun->GeneratePrimaryVertex(anEvent);

//particle 3 at vertex 3 (same as vertex 2)
//
fParticleGun->SetParticleMomentumDirection(G4ThreeVector(ux,uy,0));    
fParticleGun->SetParticleEnergy(1*GeV);
fParticleGun->GeneratePrimaryVertex(anEvent);

// randomize time zero of anEvent
//
G4double tmin = 0*s, tmax = 10*s;
G4double t0 = tmin + (tmax - tmin)*G4UniformRand();
G4PrimaryVertex* aVertex = anEvent->GetPrimaryVertex();
while (aVertex) {
  aVertex->SetT0(t0);
  aVertex = aVertex->GetNext();
}
```

With invoking `fParticleGun->GeneratePrimaryVertex(anEvent);` for several times, several particles can be generated. 

**Second method**: Generating several particles in the same vertex. ([example of freya](/media/pxy/Documents/LBNL/G4Simulation/freya/src/SingleSource.cc))

```cpp
// create a new vertex
G4PrimaryVertex* vertex = new G4PrimaryVertex(particle_position,particle_time);

for( G4int i=0; i<NumberOfParticlesToBeGenerated; i++ ) {
  // Angular stuff
  particle_momentum_direction = angGenerator->GenerateOne();
  // Energy stuff
  particle_energy = eneGenerator->GenerateOne(particle_definition);

  // create new primaries and set them to the vertex
  G4double mass =  particle_definition->GetPDGMass();
  G4double energy = particle_energy + mass;
  G4double pmom = std::sqrt(energy*energy-mass*mass);
  G4double px = pmom*particle_momentum_direction.x();
  G4double py = pmom*particle_momentum_direction.y();
  G4double pz = pmom*particle_momentum_direction.z();

  G4PrimaryParticle* particle =
    new G4PrimaryParticle(particle_definition,px,py,pz);
  particle->SetMass( mass );
  particle->SetCharge( particle_charge );
  particle->SetPolarization(particle_polarization.x(),
                particle_polarization.y(),
                particle_polarization.z());
  // Set bweight equal to the multiple of all non-zero weights
  particle_weight = biasRndm->GetBiasWeight();
  // pass it to primary particle
  particle->SetWeight(particle_weight);

  vertex->SetPrimary( particle );
}
// now pass the weight to the primary vertex. CANNOT be used here!
//  vertex->SetWeight(particle_weight);
evt->AddPrimaryVertex( vertex );
```

Defining only one vertex and adding several particles into the vertex can help generating several particles in the same position.

### The method of getting primary particles information

The method can be invoked in `PrimaryGeneratorAction::GeneratePrimaries(G4Event* anEvent)`.

```cpp
for (G4int i = 0; i < anEvent->GetNumberOfPrimaryVertex();i++){
  G4PrimaryVertex* vertex = anEvent->GetPrimaryVertex(i);
  for (G4int j = 0; j < vertex->GetNumberOfParticle(); j++){
    G4PrimaryParticle* particle = vertex->GetPrimary(j);
    fHistoManager->UpdateSource(particle->GetPDGcode(),particle->GetKineticEnergy(),vertex->GetT0(),
                                vertex->GetX0(),vertex->GetY0(),vertex->GetZ0(),
                                particle->GetPx(),particle->GetPy(),particle->GetPz(),anEvent->GetEventID());
  }
}
```



## G4VolumeStore

One can get access to all the volumes via `G4LogicalVolumeStore` or `G4PhysicalVolumeStore`.

```cpp
#include "G4LogicalVolumeStore.hh"
#include "G4LogicalVolume.hh"
#include "G4Box.hh"

if (!fEnvelopeBox)
{
  G4LogicalVolume* envLV
    = G4LogicalVolumeStore::GetInstance()->GetVolume("Envelope");
  if ( envLV ) fEnvelopeBox = dynamic_cast<G4Box*>(envLV->GetSolid());
}

if ( fEnvelopeBox ) {
  envSizeXY = fEnvelopeBox->GetXHalfLength()*2.;
  envSizeZ = fEnvelopeBox->GetZHalfLength()*2.;
}  
```

The method of `G4PhysicalVolumeStore`:

```cpp
class G4PhysicalVolumeStore : public std::vector<G4VPhysicalVolume*>
{
  public:  // with description
  
    static void Register(G4VPhysicalVolume* pSolid);
      // Add the volume to the collection.
    static void DeRegister(G4VPhysicalVolume* pSolid);
      // Remove the volume from the collection.
    static G4PhysicalVolumeStore* GetInstance();
      // Get a ptr to the unique G4PhysicalVolumeStore, creating it if necessary.
    static void SetNotifier(G4VStoreNotifier* pNotifier);
      // Assign a notifier for allocation/deallocation of the physical volumes.
    static void Clean();
      // Delete all physical volumes from the store. Mother logical volumes
      // are automatically notified and have their daughters de-registered.

    G4VPhysicalVolume* GetVolume(const G4String& name,
                                 G4bool verbose=true) const;
      // Return the pointer of the first volume in the collection having
      // that name.

    virtual ~G4PhysicalVolumeStore();
      // Destructor: takes care to delete allocated physical volumes.

  protected:

    G4PhysicalVolumeStore();

  private:

    static G4PhysicalVolumeStore* fgInstance;
    static G4ThreadLocal G4VStoreNotifier* fgNotifier;
    static G4ThreadLocal G4bool locked;
};
```





## Sensitive Detector

`G4VSensitiveDetector` provides a method to track the trace of hit in interested volume. In `ProcessHits` method, sensitive detector creates hits from
step and store them into `HitsCollection`. `HitsCollection` will be stored in `G4HCofThisEvent` of `G4Event`. `HitsCollectionOfThisEvent` will be
accessible at the end of event.

For a `Step` in Geant4, there are two points: `PreStepPoint` and `PostStepPoint`. Actually,  In case a step is limited by a volume boundary, **the end point (PostStepPoint) physically stands on the boundary, and it logically belongs to the next volume**, which means you must get the volume information from the `PreStepPoint`. 

The method of `ProcessHits` will be invoked in case of the `PreStepPoint` is in the sensitive volume.

Each LOGICAL VOLUME can have a pointer to a sensitive detector. Then this volume becomes SENSITIVE.

The sensitive detector is defined with `ConstructSDandField` method.

```cpp
// basic strategy to define a sensitive detector
#include "MySD.hh"
#include "G4VSensitiveDetector.hh"
#include "G4SDManager.hh"

void B2bDetectorConstruction::ConstructSDandField()
{
  G4LogicalVolume* myLogCalor = ……;
  G4VSensitiveDetector* pSensitivePart = new MySD(“/mydet”);
  G4SDManager* SDMan = G4SDManager::GetSDMpointer(); // get sensitive detector management pointer
  SDMan->AddNewDetector(pSensitivePart);

  myLogCalor->SetSensitiveDetector(pSensitivePart); // 1st method to register sd
  SetSensitiveDetector(myLogCalor, pSensitivePart); // 2nd method to register sd
}
```

If the detector is defined via GDML file, the sensitive detector can be defined as follows:

```cpp
void DetectorConstruction::ConstructSDandField()
{
  //------------------------------------------------ 
  // Sensitive detectors
  //------------------------------------------------ 

  G4SDManager* SDman = G4SDManager::GetSDMpointer();
  
  G4String trackerChamberSDname = "Tracker";
  G4VSensitiveDetector* aTrackerSD = new MySD(trackerChamberSDname);
  SDman->AddNewDetector( aTrackerSD );
   
  // The same as above, but now we are looking for
  // sensitive detectors setting them for the volumes
  const G4GDMLAuxMapType* auxmap = fParser.GetAuxMap();
  for(G4GDMLAuxMapType::const_iterator iter=auxmap->begin(); iter!=auxmap->end(); iter++) 
  {
    G4cout << "Volume " << ((*iter).first)->GetName()
           << " has the following list of auxiliary information: "
           << G4endl << G4endl;
    for (G4GDMLAuxListType::const_iterator vit=(*iter).second.begin();
         vit!=(*iter).second.end();vit++)
    {
      if ((*vit).type=="SensDet")
      {
        G4cout << "Attaching sensitive detector " << (*vit).value
               << " to volume " << ((*iter).first)->GetName()
               <<  G4endl << G4endl;

        G4VSensitiveDetector* mydet = 
          SDman->FindSensitiveDetector((*vit).value);
        if(mydet) 
        {
          G4LogicalVolume* myvol = (*iter).first;
          myvol->SetSensitiveDetector(mydet);
        }
        else
        {
          G4cout << (*vit).value << " detector not found" << G4endl;
        }
      }
    }
  }
}
```



One can store various types information by implementing your own concrete Hit class. A `G4Event` object has a `G4HCofThisEvent` object at the
end of (successful) event processing. `G4HCofThisEvent` object stores all hits collections made within the event. The following is an [example](/media/pxy/Documents/LBNL/G4Simulation/Pu239).

```cpp
// implementation of Hit, MyHit.hh, example from Pu239
#ifndef MyHit_h
#define MyHit_h 1

#include "G4VHit.hh"
#include “G4THitsCollection.hh”
#include "G4Allocator.hh"
class MyHit : public G4VHit
{
  public:
    MyHit();
    MyHit(G4int CopyNo);
    MyHit(const MyHit&);
    MyHit(some_user_defined_arguments);
  virtual ~MyHit();

  // operators
  const MyHit& operator=(const MyHit&);
  G4int operator==(const MyHit&) const;

  inline void *operator new(size_t);
  inline void operator delete(void *aHit);

  // methods from base class
  // virtual void Draw();
  virtual void Print();
  
  public:
  // some set methods
  void SetTrackID   (G4int value)    { fTrackID = value; }; 
  void SetEDep      (G4double value) { fEDep = value; }; 
  // some get methods
  G4int    GetTrackID()    { return fTrackID; }; 
  G4double GetEDep()       { return fEDep; }; 
  
  private:
  // some data members
  G4int fVolumeCopyNumber;
  G4int    fTrackID;
  G4double fEDep;
};

typedef G4THitsCollection<MyHit> MyHitsCollection;

extern G4ThreadLocal G4Allocator<MyHit>* MyHitAllocator;

inline void* MyHit::operator new(size_t)
{
    if (!MyHitAllocator)
        MyHitAllocator = new G4Allocator<MyHit>;
    return (void*)MyHitAllocator->MallocSingle();
}

inline void MyHit::operator delete(void* hit)
{
    MyHitAllocator->FreeSingle((MyHit*) hit);
}

#endif
```

The implementation of `MyHit.cc`:

```cpp
#include "MyHit.hh"

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

G4ThreadLocal G4Allocator<MyHit>* MyHitAllocator=0;

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

MyHit::MyHit()
: G4VHit(),fVolumeCopyNumber(0)
{
    fTrackID = 0;
    fEDep = 0.;
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

MyHit::~MyHit()
{}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

MyHit::MyHit(G4int CopyNo)
: G4VHit(),fVolumeCopyNumber(CopyNo)
{
    fTrackID = 0;
    fEDep = 0.;
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

MyHit::MyHit(const MyHit &right)
: G4VHit() {
    fVolumeCopyNumber   = right.fVolumeCopyNumber;
    fTrackID    = right.fTrackID;
    fEDep       = right.fEDep;
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

const MyHit& MyHit::operator=(const MyHit& right)
{
    fVolumeCopyNumber   = right.fVolumeCopyNumber;
    fTrackID    = right.fTrackID;
    fEDep       = right.fEDep;
  
    return *this;
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

G4int MyHit::operator==(const MyHit &right) const
{
    return ( this == &right ) ? 1 : 0;
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

void MyHit::Print()
{
  G4cout
     << "  VolumeNo: "   << std::setw(2) << fVolumeCopyNumber
     << "  TrackID: "    << std::setw(2) << fTrackID
     << "  Edep: "       << std::setw(4) << std::setprecision(3) << G4BestUnit(fEDep,"Energy")
     << G4endl;
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

```

With the previous implementation of `MyHit`, one can set the trackID and deposited energy in each hit. Here `fVolumeCopyNumber` was used to represent volumes with different copy number.

The implementation of `MySD.hh`:

```cpp
/// \file B2TrackerSD.hh
/// \brief Definition of the B2TrackerSD class

#ifndef MySD_h
#define MySD_h 1

#include "G4VSensitiveDetector.hh"
#include "MyHit.hh"
#include <vector>

class G4Step;
class G4HCofThisEvent;
class G4TouchableHistory;

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

/// B2Tracker sensitive detector class
///
/// The hits are accounted in hits in ProcessHits() function which is called
/// by Geant4 kernel at each step. A hit is created with each step with non zero 
/// energy deposit.

class MySD : public G4VSensitiveDetector
{

public:
    MySD(G4String name);
    virtual ~MySD();

    virtual void Initialize(G4HCofThisEvent*HCE);
    virtual G4bool ProcessHits(G4Step*aStep,G4TouchableHistory*ROhist);
    virtual void   EndOfEvent(G4HCofThisEvent* hitCollection);

private:
    MyHitsCollection* fHitsCollection;
    G4int fHCID;
};

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

#endif
```

The method of `ProcessHits` is used to create hit process.

The implementation of `MySD.cc`:

```cpp
// $Id: B2TrackerSD.cc 87359 2014-12-01 16:04:27Z gcosmo $
//
/// \file B2TrackerSD.cc
/// \brief Implementation of the B2TrackerSD class

#include "MySD.hh"
#include "MyHit.hh"

#include "G4LogicalVolumeStore.hh"
#include "G4LogicalVolume.hh"
#include "G4VProcess.hh"

#include "G4HCofThisEvent.hh"
#include "G4TouchableHistory.hh"
#include "G4Track.hh"
#include "G4Step.hh"
#include "G4SDManager.hh"
#include "G4ios.hh"

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

MySD::MySD(G4String name)
: G4VSensitiveDetector(name), fHitsCollection(NULL), fHCID(-1)
{
    collectionName.insert("MyColl");
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

MySD::~MySD()
{}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

void MySD::Initialize(G4HCofThisEvent* hce)
{
    // Create hits collection
    fHitsCollection
        = new MyHitsCollection(SensitiveDetectorName,collectionName[0]);// Instantiate hits collection(s) 

    // Add this collection in hce
    if (fHCID<0)
        fHCID = G4SDManager::GetSDMpointer()->GetCollectionID(collectionName[0]); //Get the unique ID number for this collection.
    hce->AddHitsCollection(fHCID,fHitsCollection);  //attach hits collection(s) to G4HCofThisEvent object given in the argument.
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

G4bool MySD::ProcessHits(G4Step*step, G4TouchableHistory*ROhist)
{
    G4LogicalVolume* LogicSD = step->GetPreStepPoint()->GetPhysicalVolume()->GetLogicalVolume(); // the volume of current step
    
    if ( LogicSD->GetMaterial()->GetName() == "NaI" ){
        G4int copyNo = 0;
        if ( LogicSD->GetName() == "BTNaI" ) {
            G4TouchableHistory* touchable
                = (G4TouchableHistory*)(step->GetPostStepPoint()->GetTouchable()); // touchable is the same as ROhist (I think)
            G4VPhysicalVolume* physical = ROhist->GetVolume();
            copyNo = physical->GetCopyNo();
            copyNo = ROhist-> GetReplicaNumber(); // same as previous line
        }
        else if ( LogicSD->GetName() == "SideNaI" )
            copyNo = 2;
        else
            G4cout << "Error, check the logical volume...." << G4endl;

        // MyHit* hit = (*fHitsCollection)[copyNo];
        MyHit* hit = new MyHit(copyNo); // implementation of Myhit

        hit->SetTrackID(step->GetTrack()->GetTrackID());
        hit->SetEDep(step->GetTotalEnergyDeposit());
      
        fHitsCollection->insert( hit ); // add new hit to fHitsCollection

        return true;
    }
    
    G4cout<<"Error, the hit is not ...."<<LogicSD->GetName()<<G4endl;
    return false;
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

void MySD::EndOfEvent(G4HCofThisEvent*)
{
  if ( verboseLevel>1 ) { 
     G4int nofHits = fHitsCollection->entries();
     G4cout << G4endl
            << "-------->Hits Collection: in this event they are " << nofHits 
            << " hits in the tracker chambers: " << G4endl;
     for ( G4int i=0; i<nofHits; i++ ) (*fHitsCollection)[i]->Print();
  }
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

```

This ProcessHits() method is invoked for every steps in the volume(s) where this sensitive detector is assigned. The basic process is the implementation of `MyHit` and inserting the `hit` to `fHitsCollection`.

The method of `EndOfEvent` is invoked at the end of processing an event. It is invoked even if the event is aborted. It is invoked before `UserEndOfEventAction`.

Get hit collection in `EndOfEventAction`:

```cpp
void EventAction::EndOfEventAction(const G4Event* event)
{
  static int fHCID = -1; 
  if(fHCID<0)
    fHCID = G4SDManager::GetSDMpointer()->GetCollectionID("Tracker/MyColl");

  G4HCofThisEvent* hce = event->GetHCofThisEvent(); // Get hit collection
  if (!hce)
  {
      G4ExceptionDescription msg;
      msg << "No hits collection of this event found." << G4endl;
      G4Exception("EventAction::EndOfEventAction()",
                  "HCCode001", JustWarning, msg);
      return;
  }

  // Get hits collections and print hit information
  MyHitsCollection* hHCCo
    = static_cast<MyHitsCollection*>(hce->GetHC(fHCID));

  if (hHCCo){
    G4int n_hit = hHCCo->entries();
    if (verbosityLevel>0)
      G4cout << "Number of hits: " << n_hit << "." <<G4endl;
    for (G4int i=0;i<n_hit;i++){
      MyHit* aHit = (*hHCCo)[i];
      if (verbosityLevel>0)
        aHit->Print();
      fHistoManager->UpdateEnergy(aHit->GetVID(),   aHit->GetPID(),
                                  aHit->GetEDep(),  
                                  G4RunManager::GetRunManager()->GetCurrentRun()->GetNumberOfEventToBeProcessed(),
                                  event->GetEventID(),  aHit->GetX(), aHit->GetY(), aHit->GetZ());
    }
  }
  fHistoManager->FillEnergy();

  if ( event->GetEventID() % PrintNumber == PrintNumber - 1 ) {
    time(&finish);
    G4cout << "      Events of " << PrintNumber << " simulated; time cost: "
           << std::setw(6) << std::setprecision(4) << difftime(finish, start) 
           << "secs." << G4endl;
  }
}
```



In the previous method, each step in interested volume represents a hit. Another [example](/media/pxy/Documents/LBNL/G4Simulation/muonVis) shows how to define only one hit in the total event.

```cpp
// PMTSD.cc
void PMTSD::Initialize(G4HCofThisEvent* hce)
{
    fHitsCollection
      = new PMTHitsCollection(SensitiveDetectorName,collectionName[0]);
    if (fHCID<0)
    { fHCID = G4SDManager::GetSDMpointer()->GetCollectionID(fHitsCollection); }
    hce->AddHitsCollection(fHCID,fHitsCollection);

    // fill coincidence detectors hits with zero energy deposition
    for (G4int i=0;i<2;i++)
    {
        PMTHit* hit = new PMTHit(i);
        fHitsCollection->insert(hit);
    }
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

G4bool PMTSD::ProcessHits(G4Step*step, G4TouchableHistory*)
{
    G4Track* track = step->GetTrack();
    if (track->GetDefinition()->GetParticleName() == "opticalphoton"){
      // get volume number (0:up, 1:down)
      G4TouchableHistory* touchable
        = (G4TouchableHistory*)(step->GetPostStepPoint()->GetTouchable());
      G4VPhysicalVolume* physical = touchable->GetVolume();
      G4int copyNo = physical->GetCopyNo();

      // obtain photon energy and time
      G4double e = track->GetKineticEnergy();
      G4double t = track->GetGlobalTime();

      PMTHit* hit = (*fHitsCollection)[copyNo];
      // add energy deposition
      hit->RecordEdepAndTime(e, t);

      return true;
    }
    return false;
}
```

Here, the `PMTHit` is initialized in `Initialize` rather than `ProcessHits`. **The `Initialize` will be invoked at the start of each event.**



## Physics List

see http://geant4.cern.ch/collaboration/working_groups.shtml and http://geant4.slac.stanford.edu/MSFC2012/ChoosePhys.pdf.



## G4StackingAction
[reference here](https://geant4.web.cern.ch/geant4/UserDocumentation/UsersGuides/ForApplicationDeveloper/html/ch06s02.html)

This class has three virtual methods, `ClassifyNewTrack`, `NewStage` and `PrepareNewEvent` which the user may override in order to control the various track stacking mechanisms. ExampleN04 could be a good example to understand the usage of this class.

`ClassifyNewTrack()` is invoked by `G4StackManager` whenever a new `G4Track` object is "pushed" onto a stack by `G4EventManager`. `ClassifyNewTrack()` returns an enumerator, `G4ClassificationOfNewTrack`, whose value indicates to which stack, if any, the track will be sent. This value should be determined by the user. `G4ClassificationOfNewTrack` has four possible values:

`fUrgent` - track is placed in the *urgent* stack

`fWaiting` - track is placed in the *waiting* stack, and will not be simulated until the *urgent* stack is empty

`fPostpone` - track is postponed to the next event

`fKill` - the track is deleted immediately and not stored in any stack.

These assignments may be made based on the origin of the track which is obtained as follows:

`G4int ParentID = aTrack->GetParentID();`

where

`ParentID = 0` indicates a primary particle

`ParentID > 0` indicates a secondary particle

`ParentID < 0` indicates postponed particle from previous event. 

 `NewStage()` is invoked when the *urgent* stack is empty and the *waiting* stack contains at least one `G4Track` object. Here the user may kill or re-assign to different stacks all the tracks in the *waiting* stack by calling the `stackManager->ReClassify()` method which, in turn, calls the`ClassifyNewTrack()` method. If no user action is taken, all tracks in the *waiting* stack are transferred to the *urgent* stack. The user may also decide to abort the current event even though some tracks may remain in the *waiting* stack by calling `stackManager->clear()`. This method is valid and safe only if it is called from the `G4UserStackingAction` class. A global method of event abortion is

`G4UImanager * UImanager = G4UImanager::GetUIpointer();`

`UImanager->ApplyCommand("/event/abort");`

`PrepareNewEvent()` is invoked at the beginning of each event. At this point no primary particles have been converted to tracks, so the *urgent* and *waiting* stacks are empty. However, there may be tracks in the *postponed-to-next-event* stack; for each of these the `ClassifyNewTrack()` method is called and the track is assigned to the appropriate stack.

关于StackingAction中各methods的调用时间, 可以参考[ShortCourse](http://geant4.in2p3.fr/2005/Workshop/ShortCourse/session1/M.Asai.pdf), [Downloaded](/media/pxy/Documents/RadLab/理论推导及参考资料/Geant4_参考资料/M.Asai.pdf).

`ClassifyNewTrack()` method decides which stack each newly storing track to be stacked (or to be killed). **By default, all tracks go to Urgent stack.**
The tracks in Urgent stack will be processed first. Once Urgent stack becomes empty, all tracks in Waiting stack are transferred to Urgent stack. **After all tracks in Waiting stack are tansferred, the NewStage() method is invoked.** All tracks which had been transferred from Waiting stack to Urgent stack can be reclassified by invoking `stackManager->ReClassify()`. The method of `PrepareNewEvent()` will be invoked at the beginning of each event for resetting the classification scheme.

Some interesting tips of stacking manipulations.

1. Classify all secondaries as fWaiting until Reclassify() method is invoked.

   You can simulate all primaries before any secondaries.

   For example, with the following setting, all primaries will be simulated first.

```cpp
G4ClassificationOfNewTrack
StackingAction::ClassifyNewTrack(const G4Track * aTrack)
{
    if (aTrack->GetParentID()>0)
        return fWaiting;
    else 
        return fUrgent;
}
```



2. Suspend a track on its fly. Then this track and all of already generated secondaries are pushed to the stack.

  Given a stack is " Given a stack is last-in-first-out”, secondaries are tracked prior to the original suspended track.

  Quite effective for Cherenkov lights
3. Suspend all tracks that are leaving from a region, and classify these suspended tracks as fWaiting until Reclassify() method is invoked.

  You can simulate all tracks in this region prior to other regions.

  Note that some back splash tracks may come back into this region later.

4. Classify tracks below a certain energy as fWaiting until Reclassify() method is invoked.

  You can simulate the event roughly before being bothered by low energy EM showers

对于一次event中多个源粒子的情况, stack中按照**last-in-first-out**的原则, trackid最大的粒子先process, 最后才是trackid=1的粒子. 但是按照入栈的顺序, 实际在产生secondary的时候仍是trackid=1的粒子先产生, 在所有的track都写入stack之后, 才会执行后续的过程.


## G4OpBoundaryProcess

This part shows how `G4OpBoundaryProcess` works. Reference: `G4OpBoundaryProcess.cc`.

```cpp
G4VParticleChange*
G4OpBoundaryProcess::PostStepDoIt(const G4Track& aTrack, const G4Step& aStep)
{
        theStatus = Undefined;
  
        G4bool isOnBoundary =
                (pStep->GetPostStepPoint()->GetStepStatus() == fGeomBoundary);

        if (isOnBoundary) {
           Material1 = pStep->GetPreStepPoint()->GetMaterial();
           Material2 = pStep->GetPostStepPoint()->GetMaterial();
        } else {
           theStatus = NotAtBoundary;
           if ( verboseLevel > 0) BoundaryProcessVerbose();
           return G4VDiscreteProcess::PostStepDoIt(aTrack, aStep);
        }

      if (aTrack.GetStepLength()<=kCarTolerance/2){
              theStatus = StepTooSmall;
                  if ( verboseLevel > 0) BoundaryProcessVerbose();
              return G4VDiscreteProcess::PostStepDoIt(aTrack, aStep);
      }

      G4MaterialPropertiesTable* aMaterialPropertiesTable;
          G4MaterialPropertyVector* Rindex;

      aMaterialPropertiesTable = Material1->GetMaterialPropertiesTable();
          if (aMaterialPropertiesTable) {
          Rindex = aMaterialPropertiesTable->GetProperty("RINDEX");
      }
      else {
              theStatus = NoRINDEX;
                  if ( verboseLevel > 0) BoundaryProcessVerbose();
                  aParticleChange.ProposeLocalEnergyDeposit(thePhotonMomentum);
          aParticleChange.ProposeTrackStatus(fStopAndKill);
          return G4VDiscreteProcess::PostStepDoIt(aTrack, aStep);
      }

          if (Rindex) {
          Rindex1 = Rindex->Value(thePhotonMomentum);
      }
      else {
              theStatus = NoRINDEX;
                  if ( verboseLevel > 0) BoundaryProcessVerbose();
                  aParticleChange.ProposeLocalEnergyDeposit(thePhotonMomentum);
          aParticleChange.ProposeTrackStatus(fStopAndKill);
          return G4VDiscreteProcess::PostStepDoIt(aTrack, aStep);
      }

      theReflectivity =  1.;	// 默认反射率 1.0
      theEfficiency   =  0.;	// 默认效率   0
      theTransmittance = 0.;

      theSurfaceRoughness = 0.;		//默认表面粗糙度 0

      theModel = glisur;			// 默认model及表面
      theFinish = polished;

      G4SurfaceType type = dielectric_dielectric;

      Rindex = NULL;
      OpticalSurface = NULL;

      G4LogicalSurface* Surface = NULL;

      Surface = G4LogicalBorderSurface::GetSurface(thePrePV, thePostPV); // 首先从Physical volume中寻找optical surface

      if (Surface == NULL){	// 如果G4LogicalBorderSurface中没有设置surface,尝试从G4LogicalSkinSurface中获取surface
        G4bool enteredDaughter= (thePostPV->GetMotherLogical() ==
                                 thePrePV ->GetLogicalVolume());
        if(enteredDaughter){
          Surface = 
                G4LogicalSkinSurface::GetSurface(thePostPV->GetLogicalVolume());
          if(Surface == NULL)
            Surface =
                  G4LogicalSkinSurface::GetSurface(thePrePV->GetLogicalVolume());
        }
        else {
          Surface =
                G4LogicalSkinSurface::GetSurface(thePrePV->GetLogicalVolume());
          if(Surface == NULL)
            Surface =
                  G4LogicalSkinSurface::GetSurface(thePostPV->GetLogicalVolume());
        }
      }

      if (Surface) OpticalSurface = 
             dynamic_cast <G4OpticalSurface*> (Surface->GetSurfaceProperty());

      if (OpticalSurface) {

             type  = OpticalSurface->GetType();
         theModel  = OpticalSurface->GetModel();
         theFinish = OpticalSurface->GetFinish();
```

The previous code shows how the parameters are passed to `PostStepDoIt`. The default properties are as follows:

```cpp
theReflectivity =  1.;	// 默认反射率 1.0
theEfficiency   =  0.;	// 默认效率   0
theTransmittance = 0.;

theSurfaceRoughness = 0.;		//默认表面粗糙度 0

theModel = glisur;			// 默认model及表面
theFinish = polished;

G4SurfaceType type = dielectric_dielectric;

Rindex = NULL;
OpticalSurface = NULL;

G4LogicalSurface* Surface = NULL;
```

### REFLECTIVITY

If reflectivity property is defined, the value will be used. If not, the reflectivity will be calculated depending on incident angle, polarization and complex refractive. I think the equation is equal to [the formula](https://en.wikipedia.org/wiki/Fresnel_equations#cite_note-FOOTNOTEHecht1987100-1). (not quite sure, but basically correct).

```cpp

PropertyPointer =
        aMaterialPropertiesTable->GetProperty("REFLECTIVITY");
PropertyPointer1 =
        aMaterialPropertiesTable->GetProperty("REALRINDEX");
PropertyPointer2 =
        aMaterialPropertiesTable->GetProperty("IMAGINARYRINDEX");

iTE = 1;
iTM = 1;

if (PropertyPointer) {

   theReflectivity =
            PropertyPointer->Value(thePhotonMomentum); // G4PhysicsVector::Value() method, 插值获取相应momentum下的reflectivity.

} else if (PropertyPointer1 && PropertyPointer2) {

   CalculateReflectivity();

}



void G4OpBoundaryProcess::CalculateReflectivity()
{
  G4double RealRindex =
           PropertyPointer1->Value(thePhotonMomentum);
  G4double ImaginaryRindex =
           PropertyPointer2->Value(thePhotonMomentum);

  // calculate FacetNormal
  if ( theFinish == ground ) {
     theFacetNormal =
               GetFacetNormal(OldMomentum, theGlobalNormal);
  } else {
     theFacetNormal = theGlobalNormal;
  }

  G4double PdotN = OldMomentum * theFacetNormal;
  cost1 = -PdotN;

  if (std::abs(cost1) < 1.0 - kCarTolerance) {
     sint1 = std::sqrt(1. - cost1*cost1);
  } else {
     sint1 = 0.0;
  }

  G4ThreeVector A_trans, A_paral, E1pp, E1pl;
  G4double E1_perp, E1_parl;

  if (sint1 > 0.0 ) {
     A_trans = OldMomentum.cross(theFacetNormal);
     A_trans = A_trans.unit();
     E1_perp = OldPolarization * A_trans;
     E1pp    = E1_perp * A_trans;
     E1pl    = OldPolarization - E1pp;
     E1_parl = E1pl.mag();
  }
  else {
     A_trans  = OldPolarization;
     // Here we Follow Jackson's conventions and we set the
     // parallel component = 1 in case of a ray perpendicular
     // to the surface
     E1_perp  = 0.0;
     E1_parl  = 1.0;
  }

  //calculate incident angle
  G4double incidentangle = GetIncidentAngle();

  //calculate the reflectivity depending on incident angle,
  //polarization and complex refractive

  theReflectivity =
             GetReflectivity(E1_perp, E1_parl, incidentangle,
                                                 RealRindex, ImaginaryRindex);
}
```

