# Documentation for ConnectXL: the CSXL's AI Assistant
> Developed by Team B1: [Caroline Bryan](https://github.com/cgbryan1), [Katie Brown](https://github.com/kgbrown5), [Emma Coye](https://github.com/emmacoye), & [Manasi Chaudhary](https://github.com/mchaudh-21)
> 
> COMP423: Foundations of Software Engineering Final Project

## Overview
Currently, there is no way to find classmates or peers in the CSXL, and all reservations must be adjusted manually. Users unfamiliar with the structure of the CSXL may struggle to manage reservations and be unaware of CSXL resources, such as the ambassadors, that can offer help. With many students utilizing the CSXL for coworking and room reservations, there is a need for an on-demand, virtual assistant.

A virtual assistant would allow users to conversationally query real-time usage information and cancel reservations in natural language. Our feature empowers CSXL users to manage their reservations with ease and facilitates coworking by allowing users to check the status of friends in the CSXL.



## Adding the Chatbot

ConnectXL is implemented as a chatbot displayed on the [coworking page](https://csxl-team-b1-comp423-25s.apps.unc.edu/coworking) on the CSXL web application.

### Frontend

The chat is an Angular Widget in the Coworking module, inserted into the frontend at `frontend/src/app/coworking/coworking-home/coworking-home.component.html` and defined in `frontend/src/app/coworking/widgets/chat-bot`. The widget is an Angular Material Card with a toggleable information button (to help users get acquainted with the virtual assistant), a chat display to track messages, and Material Form Input to write messages. The chat display is a CDK Virtual Scroll Viewport (documentation found [here](https://material.angular.dev/cdk/scrolling/overview)) that populates messages as they are added to a cache and messages are styled based on whether they come from the user or the virtual assistant. Once a user types out their message in the input box and hits <kbd>Enter</kbd>, the `onChatInput()` method adds their message to `messageCache`as a `UserMessage`, makes an `http.get()` request to the backend, and then parses the resulting message and adds it to the cache as an `AIMessage`. Both the `messageCache` and info button utilize `WritableSignal`'s to react and adapt the UI to user input.

### API Route

The `http.get()` request is made to the path `/api/ai_request` found in `backend/api/coworking/openai_request.py`. This API route takes in the user's prompt from the frontend and depends on the `AIRequestService`. The request handler passes the user prompt to this service and either returns the successful response or catches any errors that may have occurred in processing, before passing the final message back to where the frontend requested it.

### Service

`AIRequestService` handles all user input and is defined at `backend/services/coworking/openai_request.py`, it is dependent on `OpenAIService`, `ActiveUserService`, `ReservationService`, and information about the current registered user. The service has one method named `determine_request()` that takes in the user's input and returns a message string. This method is the first layer of the application that uses AI. A request is made to `_open_ai_svc.prompt()` with the user's prompt, the current date, and details about the service methods that are used to implement the features in this project. The OpenAI service takes in this data, chooses the service method that best matches what it believes the user is trying to achieve based on their prompt, then parses the prompt to match the input of the selected method. If no method is selected, the AI returns an empty model. Once the AI's response is received, it is tested against a series of conditionals to determine which feature service method the AI selected, then the expected input is parsed and the corresponding service method is called.

This service supports both stories in this project by passing all relevant information to the corresponding service methods.

### Models

The request service relies on the `GeneralAIResponse` model defined in `backend/models/coworking/openai_request.py`. This model is an argument for the `_open_ai_svc.prompt()` method, which requires an expected response model so that the AI's response is predictable. It has two fields: method, a string that when returned should match the selected response method; and expected_input, a string that should match structure of the parameter for the selected method (ie. a name or a date in a certain form).



## Feature 1: Find a Friend

![Find a friend](images/check_activity.png)

### Service

This feature is powered by ```ActiveUserService```.

The CSXL codebase defines a ```UserService```, ```ReservationService```, ```CourseSiteService```, and ```SectionService```. These are injected, along with the new ```OpenAIService```, into ```ActiveUserService``` to provide functionality for this story.

```check_if_active```: This method takes in a name (parsed from the user's request string) and determines if the requested user is in the CSXL. It then provides the name, as well as a list of active users and instructions, to the OpenAI service. Depending on the AI response, the method returns either the user's current location in the CSXL or a message saying the user is not present.

```get_all_active_users```: This helper method retrieves a dictionary of users and corresponding active room reservations, both as strings. This is used to support ```check_if_active```.


### Models 

```ActiveStudentResponse```: Response model for OpenAI request to find active coworkers, used in ```check_if_active()```. Contains a dictionary of users and their corresponding reservations, both as strings. Also contains a string message of the AI response in natural language.

This feature also utilizes the existing ```CourseSiteOverview```, ```TermOverview```, ```ReservationState```, and ```User``` models.

### Database

This feature relies on existing CSXL database fetching functionality and does not modify any data.



## Feature 2: Cancel Coworking Reservations

![Before cancellation](images/pre_cancel.png)

![Post cancellation](images/post_cancel.png)

### Service

This feature is powered by ```ReservationService```.

The CSXL codebase defines a ```PermissionService```, ```PolicyService```, ```OperatingHoursService```, and ```SeatService```. These are injected, along with the new ```OpenAIService```, into ```ReservationService``` to provide functionality for this story.

```determine_reservation_to_cancel```: This method takes in the date in datetime of the reservation a user is attempting to cancel. It uses the `_query_xl_reservations_by_date_for_user` method as well as a new `_query_xl_room_reservations_by_date_for_user` method (which has the exact same functionality just filtering for rooms instead of seats) and compiles all of the user's confirmed reservations into a list. The length of this list is then tested. If the list has no entries, a response is returned that there is no reservation to cancel; if the list has one entry, that entry is determined to be the one to cancel; and if there is more than one entry, the `determining_ai_helper` is called. Once a single reservation is selected, it is cancelled using the existing `change_reservation` method.

```determining_ai_helper```: This helper method takes in the date of the reservation to cancel and the list of all of the user's confirmed reservations. It provides this input to the OpenAI service, which then finds the user's reservation with the matching time and returns that reservation's id. The selected reservation is then returned to `determine_reservation_to_cancel`.

### Models

`IntAIResponse`: Response model for OpenAI request, used in ```determining_ai_helper()```. Contains the id of the reservation to cancel as an integer.

This feature also utilizes the existing ```Reservation```, ```ReservationPartial```, ```ReservationState```, and ```User``` models previously defined in the CSXL.

### Database

This feature relies on existing CSXL database functionality to cancel reservations.



## Libraries / Tools Used
Microsoft Azure OpenAI API, FastAPI, Python, Pydantic, Angular, Angular Material, Typescript, HTML, CSS
