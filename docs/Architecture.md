# JobWave

## Project description

TODO

### Users and data

This section aims to document main "entities" that exist within the project, and the data they possess, induce and produce. Entities are abstract concepts that generally represent user accounts managed by either a natural or a legal person.

#### Seasonal workers

Seasonal workers are the main users of the application. They can create an account and log into the mobile app,
which is their sole entrypoint for the application.

<details>
  <summary>"Primary" information</summary>
  <ul>
    <li>a full name (first and last names)</li>
    <li>a gender</li>
    <li>a birthdate</li>
    <li>a nationality</li>
    <li>a mobile phone number</li>
    <li>an email address</li>
    <li>a postal address</li>
    <li>a profile picture</li>
    <li>a CV (Curriculum Vitae)</li>
    <li>a small biography</li>
    <li>multiple job categories</li>
  </ul>
</details>

<details>
  <summary>Employers-oriented information</summary>
  <ul>
    <li>
      referrals: references of people who the seasonal workers have worked for (generally, a manager)
      <ul>
        <li>their full name</li>
        <li>their company</li>
        <li>the company’s postal address</li>
        <li>the person’s work email</li>
        <li>the person’s work phone number</li>
      </ul>
    </li>
    <li>
      past work experiences
      <ul>
        <li>job title</li>
        <li>job category</li>
        <li>company name</li>
        <li>company’s postal address</li>
        <li>start and end dates (down to the day)</li>
      </ul>
    </li>
    <li>
      availabilities: when and where the worker will be available for work
      <ul>
        <li>job category</li>
        <li>job title if possible</li>
        <li>start and end dates (down to the day)</li>
        <li>their geographical zone</li>
      </ul>
    </li>
  </ul>
</details>

#### Employers

An employer is a user whose purpose is to create job offers, receive applications from seasonal workers, and recruit them.

<details>
  <summary>Employer information</summary>
  <ul>
    <li>a name</li>
    <li>a description</li>
    <li>a profile picture</li>
    <li>a phone number</li>
    <li>an email address</li>
    <li>a postal address</li>
    <li>an API key</li>
    <li>a subscription tier</li>
  </ul>
</details>

As described in the list above, employers are separated into "subscription tiers". They represent the tier that the employer paid for, and higher tiers allow for more perks, which are listed below.

<details>
  <summary>Subscription tiers and perks</summary>
  <ul>
    <li>Free: limited amount of profiles with only the full name and the profile picture</li>
    <li>Silver: everything in the Free tier, plus the phone number and email address</li>
    <li>Gold: unlimited amount of profiles, with the same information as the Silver tier</li>
    <li>
      Platinum: everything in the Gold tier, with some other perks:
      <ul>
        <li>access to a chat room with the seasonal workers</li>
        <li>server-push notifications to profiles for all new job offers posted by the employer</li>
      </ul>
    </li>
  </ul>
</details>

_**Note**: The application is not responsible for the management of the subscription. It is made aware of the API key and subscription tier by undisclosed means, and the provided data is considered as fully trusted._

#### Job offers and applications

As stated before, employers can create and manage job offers.

<details>
  <summary>Composition of a job offer</summary>
  <ul>
    <li>job title</li>
    <li>job category</li>
    <li>start and end dates</li>
    <li>company</li>
    <li>salary</li>
    <li>benefits (if any, like company car, etc.)</li>
  </ul>
</details>

Seasonal workers can apply for existing job offers, and employers can review their profiles before choosing an applicant.
Employers can refuse as many profiles as they like, but can only accept one profile for a job offer.

_Note: Accepting a profile automatically refuses the other applicants and closes the job offer._

#### User reviews

Seasonal workers and employers can both leave reviews on eachother after a completed job.
The rating is out of 5, and an optional comment can be left with the review.

An employer-worker contract (from a job offer) is considered complete, and reviews can be written,
if and only if the job offer is accepted and the end date of the job offer is in the past at the time of the review.

## Architecture

Due to the project's size and specialization, it has been cut down into several microservices (MS). Each microservice follows the SRP (Single Responsibility Principle).
The main features are:
- decoupling: services are only responsible for their own task and can easily be swapped out since they interact through a third party
- overload risk reduction: services can be scaled on-demand because of their independence

### Critical and support services

The "critical" services can be seen as "top-level" or "backbone" services. They are the services (nearly) all other microservices rely on to properly function. On the other hand, "support" services are useful to several microservices for a certain purpose, and are decoupled from them in order to reduce software redundancy and increase data pooling.

The only support service in this project is an object storage (MinIO). It allows for image and file storage and management, for both the seasonal workers and the employers.

#### Critical services

The first critical service is the API gateway (also called "gateway"). Being the only entrypoint into the application, its task is to handle incoming requests and connections and redirect to the correct functional microservice.
It also ensures authentication and authorization throughout the whole application, and serves as a guard.
The API gateway is created with [KongGateway](https://konghq.com/products/kong-gateway), which is a reliable and resilient framework that fits our needs.

_Note: The gateway is not responsible for user storage and management, as this functionality is responsibility of a functional microservice. The gateway only serves as a proxy and a token verifier in the authentication process._

The second critical service is the message broker. The broker serves as a communication bridge between microservices, which allows for asynchronous and secure communication.

The message broker used in this project is Apache Kafka. Its multi-tenancy management improves the overall security by ensuring each functional microservice has its own credentials.
Moreover, microservices can leverage the consumer group principle to consume topics and use given messages for different purposes.

### Functional microservices

This section describes each "functional" microservice. A functional microservice manages one or several functionalities for the project.
Functional microservices can depend on data managed by other functional microservices, although they should never contact eachother directly.

_NB: Communication between functional microservices should only be through backbone services (such as the broker), and if possible, in an asynchronous way._

#### MS: Authentication

The authentication microservice is responsible for all user's authentication and token generation and is written using Spring Security. The microservice is purposefully detached from the Users MS in order to ensure functionality for connected users if the Authentication MS were to shut down.

Users are authenticated through the API Gateway, using a JWTs (Json Web Tokens). There are two JWTs:
- the Service Token (ST), used to authenticate the user against the service
- the Token Granting Token (TGT), used to generate new STs if needed

A few security rules are in place to ensure security at all times:
- the TGT must only be issued upon successful authentication using manual means of connection (i.e. email/password, security key, FIDO2, etc.)
- the TGT should be set with a medium expiry time (e.g. one or two days)
- a new ST must be issued only if:
  - the current ST has expired
  - there is no existing ST for this user, or the user is unable to provide one
- a new ST must be issued only through:
  - completion of a successful manual authentication
  - presence of a valid, not outdated, TGT provided by the user
- all STs must be set with a rationally small expiry time (e.g. a maximum of several hours) in order to mitigate token replication/guessing

#### MS: Users

TODO

#### MS: Offers

TODO

#### MS: Reviews

TODO

#### MS: Chats

TODO

#### MS: Recommendations

Seasonal workers get job offers recommended to them using a recommendation system. They can choose several job categories to influence the system.

The recommendations should be based on:
- recency of the job offer
- relevant job categories found in the offer
- company rating
- company subscription level

#### MS: Notifications

TODO

## Choices and decisions

As the project’s specifications are somewhat incomplete, we must make choices in order for things to be disambiguated.
Thus:

- Some information about seasonal workers is hidden from employers until the worker applies for a job offer: contact info (for Free employers), ratings and comments, referrals, experiences, availabilities
- The seasonal worker can, at any given time, choose the preferred job categories for them
- The chat system is the responsibility of its own micro-service, and messages are passed through a message broker and
  stored in the MS’s database
- Messages sent through the chat system are immutable and undeletable
- A seasonal worker can remove his application to a job offer
- An employer can delete a job offer, which will remove all applications
- Seasonal workers can leave ratings and comments for employers which they have worked for on the app
- Employers can only leave ratings and comments on profiles of workers which they have hired before on the app
  by both parties, and said contract has ended (the end date is in the past)
- Employers can accept only one candidate for a job offer
- Chats are not linked to an application, and only one chat can exist between a given employer and a given seasonal worker

## Key security elements

The following list ensures that security in the app is upheld and ensured.

- It is impossible to undo/cancel an account deletion request
- Accounts awaiting deletion and deleted accounts are masked (invisible) and their public data, as well as their
  interactions (applications, ratings, comments, etc.), are masked too
- Job categories are immutable and cannot be edited from the app
