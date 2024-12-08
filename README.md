# Agoric-Hack-Final: Fair Pass
Fair Pass - Event Ticketing Smart Contract Documentation
Overview
This repository contains two smart contracts, EventCreator and Event, implemented using Solidity for creating, managing, and purchasing event tickets represented as ERC-721 Non-Fungible Tokens (NFTs).

The EventCreator contract allows users to create events, while the Event contract manages the lifecycle of tickets for each event, including buying, selling, and tracking ticket usage. Tickets can be resold based on specific rules set by the event organizer, and royalties can be earned on secondary sales.

Smart Contracts
There are two primary contracts:

EventCreator: A contract for creating events and deploying new event-specific contracts (Event).
Event: A contract that manages tickets for a particular event, including buying tickets, reselling, and marking tickets as used.
Prerequisites
Solidity version: >=0.7.0 <0.9.0
OpenZeppelin ERC721 contract for managing NFTs
Table of Contents
EventCreator Contract
Functions
Events
Event Contract
Functions
Modifiers
Events
Usage Example
Security Considerations
EventCreator Contract
Purpose
The EventCreator contract allows users to create events by deploying new Event contracts, each representing an individual event with a ticketing system.

Functions
createEvent
solidity
Copy code
function createEvent(uint _numTickets, uint _price, bool _canBeResold, uint _royaltyPercent,
                     string memory _eventName, string memory _eventSymbol) external returns(address newEvent)
Parameters:
_numTickets: Number of tickets available for the event.
_price: Price per ticket.
_canBeResold: Boolean indicating whether tickets can be resold on the secondary market.
_royaltyPercent: Percentage royalty event organizers will receive from ticket resales.
_eventName: Name of the event (for the ERC-721 token).
_eventSymbol: Symbol for the ERC-721 token.
Returns: The address of the newly created Event contract.
Description: Creates a new event contract and emits the CreateEvent event.
getEvents
solidity
Copy code
function getEvents() external view returns(Event[] memory _events)
Returns: List of all events created.
Description: Returns an array of all Event contract addresses.
Events
CreateEvent: Emitted when a new event is created.
solidity
Copy code
event CreateEvent(address _creator, address _event);
Event Contract
Purpose
The Event contract represents an individual event and allows users to purchase, resell, and use tickets. Each ticket is represented as an ERC-721 token.

Functions
constructor
solidity
Copy code
constructor(address _owner, uint _numTickets, uint _price, bool _canBeResold, uint _royaltyPercent,
            string memory _eventName, string memory _eventSymbol) ERC721(_eventName, _eventSymbol)
Parameters:
_owner: The organizer's address.
_numTickets: Total number of tickets for the event.
_price: Price per ticket.
_canBeResold: Whether tickets can be resold.
_royaltyPercent: The royalty percentage from reselling.
_eventName: Name of the event.
_eventSymbol: Symbol for the event's token.
Description: Initializes the event with the given parameters and sets up the ERC-721 token.
buyTicket
solidity
Copy code
function buyTicket() public payable requiredStage(Stages.Active)
Description: Allows a user to buy a ticket for the event if the event is in the "Active" stage and there are tickets available. The buyer's payment is transferred to the event owner.
buyTicketFromUser
solidity
Copy code
function buyTicketFromUser(uint ticketID) public payable requiredStage(Stages.Active)
Parameters: ticketID - ID of the ticket being purchased from another user.
Description: Allows a user to buy a ticket from another user if the ticket is marked as available for resale. The resale price is deducted, and royalties are transferred to the event organizer.
setTicketToUsed
solidity
Copy code
function setTicketToUsed(uint ticketID) public requiredStage(Stages.CheckinOpen) ownsTicket(ticketID)
Parameters: ticketID - ID of the ticket to mark as used.
Description: Marks the ticket as used and burns the associated token once the user has checked in to the event.
setTicketForSale
solidity
Copy code
function setTicketForSale(uint ticketID, uint resalePrice) public requiredStage(Stages.Active) ownsTicket(ticketID)
Parameters: ticketID - ID of the ticket being marked for sale, resalePrice - Price for reselling the ticket.
Description: Allows a ticket owner to list a ticket for resale if the event allows it.
withdraw
solidity
Copy code
function withdraw() public
Description: Allows users to withdraw funds, either refunds in case of event cancellation or overpayment from purchasing tickets.
setStage
solidity
Copy code
function setStage(Stages _stage) public onlyOwner returns (Stages)
Parameters: _stage - New stage for the event (e.g., Active, Cancelled).
Description: Changes the event's status (stage) and adjusts balances accordingly.
Modifiers
onlyOwner
solidity
Copy code
modifier onlyOwner() { ... }
Description: Ensures that only the event organizer (owner) can execute the function.
requiredStage
solidity
Copy code
modifier requiredStage(Stages _stage) { ... }
Description: Ensures the event is in the specified stage.
ownsTicket
solidity
Copy code
modifier ownsTicket(uint ticketID) { ... }
Description: Ensures the caller is the owner of the specified ticket.
Events
CreateTicket: Emitted when a new ticket is created.
WithdrawMoney: Emitted when a user withdraws funds.
OwnerWithdrawMoney: Emitted when the event owner withdraws funds.
TicketForSale: Emitted when a ticket is listed for resale.
TicketSold: Emitted when a ticket is sold.
TicketUsed: Emitted when a ticket is used (and burned).
Usage Example
Deploying an Event
solidity
Copy code
FairPassEventCreator creator = new FairPassEventCreator();
creator.createEvent(100, 1 ether, true, 10, "Concert", "TICKET");
Buying a Ticket
solidity
Copy code
FairPassEvent event = FairPassEvent(eventAddress);
event.buyTicket{value: 1 ether}();
Reselling a Ticket
solidity
Copy code
event.setTicketForSale(ticketID, 1.5 ether);
event.buyTicketFromUser{value: 1.5 ether}(ticketID);
Marking a Ticket as Used
solidity
Copy code
event.setTicketToUsed(ticketID);
Security Considerations
Reentrancy Attacks: Ensure external calls (e.g., transferring funds) are secure and do not allow reentrancy attacks. For example, balances should be updated before transferring funds.
Event Cancellation: Proper handling of refunds and ensuring that only valid actions are allowed during the cancellation phase.
Access Control: Use modifiers to restrict access to functions based on the caller’s role (owner or ticket owner).
Conclusion
Fair Pass offers a robust solution for creating, managing, and trading event tickets on the Ethereum blockchain. It provides flexibility for event organizers to control the ticket lifecycle while enabling secondary sales and royalty distributions.







