## Projet CLE UPAAL

** Binomial ** :
- Franco Davy Irakoze
- Jérémy Vienne

### System
 - ** Global declarations **
 > - *** distance train *** : int -> the time, the train takes to reach the crossroad, after passing the sensor.
 > - *** crossroad *** : bool -> tell us if a car is crossing, true if a car is crossing, false if not
 > - *** canPass*** : bool -> give the permission to a car to cross, false if there is train arriving and true if not.
 > - *** pass *** : broadcast chan -> tell us that the train has crossed
 > - *** arriving *** : broadcast chan -> tell the rail that the train is approaching
 > - *** stuckCar *** : broadcast chan -> tell us that a car is stuck on the crossroad
 > - *** stuckCanPass *** : the train is arriving but the stuck car still have the time to cross. This chan gives the permission to cross to the stuck car.


 - ** CAR **
 > ** Variables : **
    > - *** count *** : int -> time counter
    > - *** r *** : int -> random value between 1 and 5  to specify the time the car will take to cross         

 > ** States : **
    > - *** Start *** -> initial state
    > - *** crossing *** -> the car is crossing

 > ** Conditions : **
    > - *** Start to crossing *** :  
      `if canPass is true, we'll get a rand value r and go to the next state`
    > - *** crossing *** :  
      `We loop on crossing state while the count value is less than r and we set the crossroad value to true`
    > - *** crossing to Start *** :   
      `There are 2 ways to go back to Start: the first one, the car has already passed the road and we set crossroad to false. The second one,the train is arriving, but the stuck car still have time to cross the road, crossroad value is set to false too `

 - ** TRAIN ***
 > ** Variables :**   
  > - *** count *** : int  -> time counter
  > - *** r *** : int  ->  random value, representing the time between the sensor and the crossroad
  > - *** r2 *** : int  -> random value, representing the time, the train will take to cross.

  > ** States : **
     > - *** Start *** -> initial state
     > - *** SensorOn *** -> The train pass the sensor position.
     > - *** StopTrain *** -> Exception state when a car is stuck and has not enough time to cross. The train must be stopped.
     > - *** crossing *** -> the train is crossing

 > ** Conditions : **
    > - *** Start to SensorOn *** :  
    `The train is arriving and the time to reach the crossroad is set randomly between 20 and 30s`
    > - *** SensorOn *** :  
    `We loop on SensorOn state while the count value is lower than the random value set previously`
    > - *** SensorOn to Stop Train *** :  
    `A car is stuck and don't have enough time to cross the road, we stop the train`
    > - *** SensorOn to crossing *** :   
    `The train has passed the sensor and is reaching the crossroad. We set randomly between 10 and 20 the time the train will take to cross`  
    > - *** crossing *** :   
    `We loop on crossing while the count value is lower than the random value set previously`
    > - *** crossing to start *** :   
    `the train passed the crossroad and we mention it to the other machines by editing the broadcast "pass"`


 - ** RAIL **
 > ** Variables :**   
  > - *** count *** : int  -> time counter

  > ** States : **
     > - *** Open *** -> initial state, the rail gate is open
     > - *** CheckIfCar *** -> The train is arriving, and we check if car is crossing before closing the gate .
     > - *** CarStuck *** -> The stuck car don't have time to cross and we must stop the train.
     > - *** StuckButCanPass *** -> A car is stuck but it still has the time to cross the cross until the train arrive
     > - *** Closed *** -> the gate is close

 > ** Conditions : **
    > - *** Open to CheckIfCar *** :  
    `The train is arriving and We set the canPass to false to unable the car to cross`
    > - *** CheckIfCar to CarStuck *** :  
    ` There is stuck car on the crossroad and it has not enough time to cross, so we set the broadcast stuckCar, the train must also be stopped `

    > - *** CheckIfCar to StuckButCanPass *** :   
    `The train is arriving, but there is enough time for the stuck car to cross the road, so both of them can continue moving, the gate are open`

    > - *** StuckButCanPass to CarStuck *** :  
    `There is stuck car on the crossroad and but finally, it has not enough time to cross, so we set the broadcast stuckCar, the train must also be stopped`
    > - *** StuckButCanPass to Closed *** :   
    `The train has passed the sensor and is reaching the crossroad. Trail has checked if a car is stuckhe stuck car on the crossroad has passed. The gate will be closed right after `  
    > - *** Closed to Open *** :   
    `The Train has passed the crossroad, we set the broadcast passed and we also set the canPass to true to enable waiting cars to cross.`

    > - *** CheckIfCar to Closed *** :   
    `The train is arriving and there is no stuck car on the crossroad, so the gate can be closed.`


### Vérification
  ** Query **  
  - `E<> train.StopTrain and rail.CarStuck` -> a state exist where the train will stop and a car is stuck

  - `E<> train.crossing` -> a state exist where the train is crossing

  - `E<> train.SensorOn` -> a state exist where the train approach the crossroad

  - `E<> rail.StuckButCanPass and car.crossing and train.SensorOn` -> a state exist where a train is approching while a car is crossing the road but the train don't need to stop because the car have time to cross the road

  - `E<> rail.Closed and train.crossing` -> a state exist where the train i crossing and the gate is closed

  - `E<> rail.preOpen and not train.crossing` -> there isn't a state where the gate will open and the train is crossing

  - `A[] (car.crossing && car1.crossing) != true` only one car can cross at a time

  - `A[] train.crossing imply rail.Closed` -> trains crossing imply that the rail is closed

  - `A[] ((car.crossing || car1.crossing) && train.crossing) != true` -> while a train is crossing the car can't cross

  ** Can't verify  **

  