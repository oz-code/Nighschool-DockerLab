# Nighschool-DockerLab
a lab to show how to use Ozcode production debugger with Docker

## Getting started

### Step 1: clone our docker demo
```bash
git clone https://github.com/oz-code/Nightschool-DockerLab
cd Nighschool-DockerLab
```
This is a tiny repo that only have a docker-compose that run several micro services written in .Net Core 3.1, and they all have Ozcode Production Debugger agennt installed and ready to go.

### Step 2: Register to Ozcode Produciton Debugger
1. go to https://app.oz-code.com, register a new account
2. create application name "DockerLab"
3. Follow the installation of docker and find ```OzCode_Agent_Token``` value
4. edit ```.env``` file and enter your token.

### Step 3: Run the example
```bash
docker-compose up
```
This should download several images from DockerHub and start your local cluster

## Demo Structure
The demo is built from the several services:

| Service | Tech | Address | Description |
|--|--|--|--|
| ServicA | .Net Core 3.1 Web.API | http://localhost:5001 | manage courses shcedules
| ServicB | .Net Core 3.1 Web.API | http://localhost:5002 | manage rooms bookings
| ServicAuth | .Net Core 3.1 Web.API | http://localhost:5003 | manage auth and token generation
| ServicUI | .Net Core 3.1 Razor pages | http://localhost:5000 | UI service
| ServicSPA | Node.JS Vue | http://localhost:5080 | UI service SPA
| SEQ | SEQ | http://localhost:5050 | Log aggregation

We are going to use our Service SPA to generate some traffic and errors our cluster and by using Ozcode Production Debbuger we are going to capture  errors and add tracepoint to debug issues.

# Lab Structure

The lab is built on two main parts: Errors and Tracepoints

## Exec 1: Errors 

### 1.1 Generating a bug
1. navigate to http:://localhost:5080 to see the main screen of the application.
2. Click on the ```Course``` tab to see all avaialbe courses. you should see all courses have no schecule 
3. Start clicking the ```Schedule``` for course ```INTRO-001`` - you should see that course schedule is created correctly and that record become green
4. Contiune to resolve schedule for ```LEGAL-001``` - works amazing.
5. When resolving schedule for ```HAND-001``` you would see a alert popup telling you there is a problem.

### 1.2 Capturing the exception
1. Go to http://app.oz-code.com/ and check your Errors dashbaord, you should see two exceptions in ```Capturing mode```. If you don't see anything try to click ``Refresh`` button at top left of the screen to reload errors
2. Recreate that bug once again by resolving scheudle for ```HAND-001```
3. Refresh the Errors Dashboard table and you would see both exceptions are no ``Captured``. Select one exception and click ``Debug``

### 1.3 Recording time travel
1. on the Debug screen of one of those excpetions click ``Capture full-time travel`` button.
2. Recreate the same exception by resolving scheudle for ```HAND-001```
3. Refresh the Errors Dashboard table and you would see both exceptions are no ``Fully Captured``. Select one exception and click ``Debug``

## Exec 2: Tracepoints

### 2.1 Creating a logical error
1. Complete Exec 1, you should have both ``INTRO-001`` and ``LEGAL-001`` courses schedule resolved (critical to generate the logical issue). If you state is different, you can restart your cluster and start over.
2. Try resolve schedule for ``FLIGHT-001``, this would cause that row schedule to become yellow as there was problem in the scheduling. Rooms were assigned but no students were registered.

### 2.2 Creating a Tracepoint Debug Session
1. go to http://www.oz-code.com/tracepoints or Click the chevron icon near the ``Errors`` at the navbar and select ``Tracepoints Sessions``.
2. Create a new Tracepoint Session by clicking the ``Create +`` button
3. Use the Code explorer to find the follwoing Class: ServiceA.Controllers.CourseController
4. Goto function "ResolveSchedule".
5. Add tracepoint at line
   ```Csharp
   course.Schedule = newSchedule;
   ```
    Add the following message to the tracepoint
   ```
   Found new schedule for course {id}
   ```
6. Click Start capturing icon.
7. Try to resolve schedule for ``FLIGHT-001`` to get your first tracepoint hit.

### 2.3 Find the logcal error
1. Contiune to add more tracepoint and by resovling the schedule again and again you would be able to find the root cause of your bug.

<p>
<details>
<summary>
NEED A HINT: click here
</summary>
   Try to add tracepoints at FindAvailableRooms, try to understand why we are not able to find any rooms for the lectures.
</details>
</p>

2.4 Dynamic logs - A Whole new world
1. Goto http://localhost:5050 to open SEQ services.
2. THis service was collecting all the logs generated from all other microservices.
3. Search ```Tracepoint is not null```
4. Using our Ozcode.ProductionDebugger.Client NuGet package, we transformed all Tracepoint hits into logs entries. This can be used to create dynamic logs in your system without any code-deploy
5. Expand the Tracepoint property and click the TracepointUrl to open the exact tracepoint hit that cause that log line to be created

# License

MIT
