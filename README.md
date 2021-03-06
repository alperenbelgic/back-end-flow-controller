[![Build status](https://ci.appveyor.com/api/projects/status/450whm3p2sphqi8t?svg=true)](https://ci.appveyor.com/project/alperenbelgic/back-end-flow-controller)

# Backend Flow Controller (BFC)

## Motivation
- ability to design or view the application flow with a human readable way
- to have full control over the rest of the iceberg
- separating the API from UI ( or any other consumer )

#### Some cases I try to avoid:
- The software is not working, I do not know why. There is no message in UI or logging.
- There is a feature/fix I should apply. I cannot find the code piece should be changed.
- I really do not know how the application flows
- I cannot decide in which state the application is now. I don't understand which tables and which columns are used to decide for the current state.
- I am not sure how I should desing the API. Which web services I should call, which orders etc.
- This V2 application became legacy. We need to re-write it.

### Best Practices For BFC
- Do not consume data directly from a service or db call in actions. Write them to flow first, then use them in another action. We want to observe the IO in the system. We want our actions to be Unit Testable.

## Some Pseudo Tests:

#### Test 1: Send an event to the flow and flow changes its state.
```

var flowDefinition = new FlowDefinition()
{
  States = new List<State>()
  {
    new State()
    {
      Name = "State1",
      Events = new List<Event>()
      {
		new Event()
		{
			Name = "Event1",
			DestinationState = "State2"
		}
      }
    },
	new State()
	{
		Name = "State2"
	}
  }
};

var flowInstance = new FlowInstance()
{
 	CurrentState = "State1",
	FlowDefinition = flowDefinition
};

var result = flowInstance.SendEvent("Event1");

Assert.IsTrue(result.Suceeded);
Assert.IsEqual(flowInstance.CurrentState, "State2");
```

#### Test 2: Send an event to the flow and flow triggers an action (which is defined in flow definition)
 ```
var mockAction = new Mock<IAction>();


var flowDefinition = new FlowDefinition()
{
  States = new List<State>()
  {
    new State()
    {
      Name = "State1",
      Events = new List<Event>()
      {
        Name = "Event1",
        Actions = new List<Action>()
        {	
        	mockAction.Object
        }
      }
    }    
  }
};

var flowInstance = new FlowInstance()
{
 	CurrentState = "State1",
	FlowDefinition = flowDefinition
};

var result = flowInstance.SendEvent("Event1");
mockAction.Verify(action => action.Execute());

```

#### Test x: Action can read flow data
 ```
 public abstract class Action1 : IAction
 {
 	public string Data1 { get; set; }
 
 	public override void Execute() 
	{ 
		// Data1 will be usable here
	}
 }
 
var mockAction = new Mock<Action1>();

dynamic flowData = new ExpandoObject();
flowData.Data1 = "hello";

var flowDefinition = new FlowDefinition()
{
  States = new List<State>()
  {
    new State()
    {
      Name = "State1",
      Events = new List<Event>()
      {
        Name = "Event1",
        Actions = new List<Action>()
        {	
        	mockAction.Object
        }
      }
    }    
  }
};

var flowInstance = new FlowInstance()
{
 	CurrentState = "State1",
	FlowDefinition = flowDefinition,
	FlowData = flowData
};

var result = flowInstance.SendEvent("Event1");
Assert.IsTrue(result.IsSuccessful);
Assert.IsEqual("hello", mockAction.Object.Data1);
```

#### Test x: Action can change flow data
 ```
 public abstract class Action1 : IAction
 {
 	public string Data1 { get; set; }
 
 	public override void Execute() 
	{ 
		this.Data1 = "hi";
	}
 }
 
var mockAction = new Mock<Action1>();

dynamic flowData = new ExpandoObject();
flowData.Data1 = "hello";

var flowDefinition = new FlowDefinition()
{
  States = new List<State>()
  {
    new State()
    {
      Name = "State1",
      Events = new List<Event>()
      {
        Name = "Event1",
        Actions = new List<Action>()
        {	
        	mockAction.Object
        }
      }
    }    
  }
};

var flowInstance = new FlowInstance()
{
 	CurrentState = "State1",
	FlowDefinition = flowDefinition,
	FlowData = flowData
};

var result = flowInstance.SendEvent("Event1");
Assert.IsTrue(result.IsSuccessful);
Assert.IsEqual("hi", flowData1.Data1);
```

#### Test x: Action can add flow data
 ```
 public abstract class Action1 : IAction
 {
 	public string Data1 { get; set; }
 
 	public override void Execute() 
	{ 
		this.Data1 = "hi";
	}
 }
 
var mockAction = new Mock<Action1>();

dynamic flowData = new ExpandoObject();

var flowDefinition = new FlowDefinition()
{
  States = new List<State>()
  {
    new State()
    {
      Name = "State1",
      Events = new List<Event>()
      {
        Name = "Event1",
        Actions = new List<Action>()
        {	
        	mockAction.Object
        }
      }
    }    
  }
};

var flowInstance = new FlowInstance()
{
 	CurrentState = "State1",
	FlowDefinition = flowDefinition,
	FlowData = flowData
};

var result = flowInstance.SendEvent("Event1");
Assert.IsTrue(result.IsSuccessful);
Assert.IsEqual("hi", flowData1.Data1);
```

#### Test x: Arrived to state actions

#### Test x: Conditional self event trigger when arrived to state or another state type like: decision state

#### Test x: Action throws exception, roll back?, result?

#### Test x: Events and actions logs created?

#### Test x: Validations: Proper state name? Next state's name exists?

#### Json.Net PoC, move it later:
```
using System;
using Newtonsoft.Json.Linq;
using Newtonsoft.Json;					

public class Program
{
	public static void Main()
	{
		string json = @"{ ""A"" : ""alperen "", ""D"" : 3 }";		
		dynamic a = JObject.Parse(json);		
		dynamic b = new JObject();		
		b.A = "ads;fadsf";
		b.C = "hodor";		
		a.Merge(b, new JsonMergeSettings { MergeArrayHandling = MergeArrayHandling.Concat });	
		
		Console.WriteLine(a.ToString());
		
		var x = a.ToObject<asdf>();
		
		Console.WriteLine(x.A);
		Console.WriteLine(x.D);
		
		a.D = 5;
		
		x.D = 7;
		
		JsonConvert.PopulateObject(a.ToString(), x);
		
		
		Console.WriteLine(x.D);
	}
}

public class asdf{

	public string A;
	public int D;

}
```
