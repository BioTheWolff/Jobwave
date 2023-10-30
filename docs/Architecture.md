# JobWave

## Glossary

TODO

## Project description

TODO

### Users and data

This section aims to document main "entities" that exist within the project, and the data they possess, induce and produce. Entities are abstract concepts that generally represent user accounts managed by either a natural or a legal person.

#### Job categories

Job categories are categories that all seasonal workers' experiences, and employers' job offers, should fit into. Those categories are immutable and cannot be modified by anybody except administrators.

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

_Note: Some information about seasonal workers is hidden from employers until the worker applies for a job offer: contact info (for Free employers), ratings and comments, referrals, experiences, availabilities_

A seasonal worker can, at any given time, change its preferred job category. This choice influences the recommendation system.

A seasonal worker can also request, if needed, deletion of their account. Once confirmed, the deletion request is uncancellable and must be approved by an administrator.
Accounts awaiting deletion, and deleted accounts, are masked (invisible) and their public data, as well as their interactions (applications, ratings, comments, etc.), are masked too.
Please note that data can be stored on the application's side after the account has been deleted, according to the country's laws and regulations in effect.

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

Seasonal workers are free to apply, and remove their application, at any given time and without cooldown or delay.

_Note: Accepting a profile automatically refuses the other applicants and closes the job offer._

#### User reviews

Seasonal workers and employers can both leave reviews on eachother after a completed job.
The rating is out of 5, and an optional comment can be left with the review.

An employer-worker contract (from a job offer) is considered complete, and reviews can be written,
if and only if the job offer is accepted and the end date of the job offer is in the past at the time of the review.

#### Messaging system

As stated above, employers bearing a Platinum subscription level key can initiate text conversations with applicants of their job offers. This text conversation is called a "chat".

_Note: there can only exist one "chat" between a given seasonal worker and a given employer, and as such, "chats" are not linked to an application, but rather to the worker applying._

This messaging system defines messages as being text only (meaning no voice messages, no file attachments, nor any other undisclosed type), encoded in UTF-8 standard, and with a maximum length of 500 characters.

For better transparency between the employer and the seasonal worker, and since this is a professional setting, sent messages are uneditable and undeletable.

## Services architecture

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

The microservice is the root of trust of the application and stores all connection data required for users to authenticate.

- for employers, this includes the Employer ID (EID) and the API key.
- for seasonal workers, this includes the Seasonal Worker ID (WID), the canonical username and the password.

_NB: The canonical username is the username used upon creating the account. It never changes, and is only used for connecting to the application._

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

The Users microservice stores and manages all informational/personal data about users (meaning all data except account data - see the [authentication microservice](#ms-authentication) for more information). It is written in NestJS to allow for easy integration with databases and simple authorization management.

As there are two types of users, we use a database for each user type:
- the employers are stored on a PostgreSQL database, for its relational schema and ease of deployment
- the seasonal workers are stored on a MongDB instance, because the data volume per user can grow rapidly, and workers' information can heavily differ from one to another

The microservice also interacts with the OSS (Object Storage Service), in this case MinIO. This is used to store profile images, and various documents (mostly PDF files) for all users.

#### MS: Offers

The Offers microservice manages the job offers and everything that revolves around it. It is responsible for the management of the job offers themselves, the applications for said job offers, and the resolution of a job offer.

The job offers and their related applications are stored in a PostgreSQL instance. The microservice is developed using Kotlin.

A job offer can be resolved into one of these two states:
- Cancelled: the job offer was cancelled by the employer
- Completed: an applicant has been chosen by the employer

If the offer is Cancelled, all current applications are thus denied.
If the offer is Completed, the microservice denies all applications automatically, except for the selected applicant, which is granted this job.

When a job offer is resolved, notifications are sent according to the resolution state. As it is not this microservice's responsibility, please see the [notifications microservice](#ms-notifications) for more information.

#### MS: Reviews

In order for seasonal workers to determine if they want to work for an employer, and for employers to determine if a seasonal worker is a good fit, both can leave reviews. This is called R&R ("Reviews and Ratings").

This microservice is in charge of managing reviews and comments that can be left by both parties.
It also manages the authorizations associated with the reviews, so the appropriate level of information is sent back depending on the client who requests it (i.e. employers depending on their subscription tier).

The R&Rs are stored in a PostgreSQL instance. The microservice also uses Deno, which is an easy, fast and cloud-ready programming language.

Please see the [appropriate section](#job-offers-and-applications) for reviews conditions and requirements.

#### MS: Chats

The Chats' microservice is responsible for the storage of all conversations and messages exchanged between employers and seasonal workers. It directly receives the messages sent, as there is no "middleman" service/microservice.
This is designed so that the microservice is the sole manager of conversations, and the microservice being shut down halts the functionality entirely.

The microservice stores messages in a PostgreSQL instance, which is the database of choice when data length is known, and the amount of data will grow steadily. This microservice also uses Deno, for the same purposes as the [R&R microservice](#ms-reviews).

#### MS: Recommendations

This microservice aggregates information about job offers and seasonal workers in order to provide accurate recommendations tailored to the workers' desires.
To accurately recommend offers, it is dependent on both the [Offers](#ms-offers) and the [R&R](#ms-reviews) microservices.

Using the broker as primary means of information gathering, the microservice collects any updates deemed interesting about:
- offers (creation, update, resolution)
- reviews (creation, deletion)
- users (preferred job category update)

The microservice is made using Deno, connected to a PostgreSQL database.

The recommendations should be based on:
- recency of the job offer
- relevant job categories found in the offer
- company rating
- company subscription level

#### MS: Notifications

In order for notifications to be sent to seasonal workers, we set up a dedicated microservice. It is written in Spring Webflux and pulls data from the message broker.

It is used to send notifications in the following cases:
- a job offer is resolved, thus accepting or denying the seasonal worker's application, if it has applied to said offer
- a job offer is created, meaning the offer should be sent to all seasonal workers (provided the employer has the correct subscription level, see [employers](#employers))
- a message is sent to the seasonal worker

## Communication: flow and security

### Data trust: root and propagation

TODO: auth as RoT, replication across MS for WID/EID, JWT trust

### Inter-service communication

TODO: message broker and topics

### External communication

TODO: api gateway and redirections
