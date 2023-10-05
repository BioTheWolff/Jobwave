# JobWave - Architecture

## Project description

description

### Seasonal workers

Seasonal workers are the main users of the application. They can create an account and log into the mobile app, which is
their sole entrypoint for the application.

Their account is composed of “primary” information:

- a full name (first and last names)
- a gender
- a birthdate
- a nationality
- a mobile phone number
- an email address
- a postal address
- a profile picture
- a CV (Curriculum Vitae)
- a small biography
- multiple job categories

Seasonal workers can also add and edit different information for the employers to see:

- referrals: references of people who the seasonal workers have worked for (generally, a manager)
  - their full name
  - their company
  - the company’s postal address
  - the person’s work email
  - the person’s work phone number
- past work experiences
  - job title
  - job category
  - company name
  - company’s postal address
  - start and end dates (down to the day)
- availabilities: when and where the worker will be available for work
  - job category
  - job title if possible
  - start and end dates (down to the day)
  - their geographical zone

### Employers

Employers have access to the application through an API. An employer is described by:

- a name
- a description
- a profile picture
- a phone number
- an email address
- a postal address
- an API key
- a subscription tier

There are 4 subscription levels:

- Free: limited amount of profiles with only the full name and the profile picture
- Silver: same as Free, with also phone number and email
- Gold: unlimited amount of profiles, with the same information as the Silver tier
- Platinum: everything in the Gold tier, with some other perks:
  - access to a chat room with the seasonal workers
  - server-push notifications to profiles for all new job offers posted by the employer

The application is not responsible for the management of the subscription. It only manages grants and permissions depending on the subscription tier, which it is aware of by undisclosed means.

### Reviews and comments

Seasonal workers and employers can leave reviews on each-other after a completed job.
The rating is out of 5, and an optional comment can be left with the review.

An employer-worker contract (from a job offer) is considered complete, and reviews can be written, if and only if the job offer is accepted and the end date of the job offer is in the past.

### Recommendation system

for seasonal workers:

- identify key experiences of the worker
- ask for which categories are preferred for the moment

TODO

### Job offers and applications

Employers can post job offers. A job offer is composed of:

- job title
- job category
- start and end dates
- company
- salary
- benefits (if any, like company car, etc.)

Seasonal workers can apply for job offers, and employers can review their profiles before choosing a candidate. Employers can refuse as many profiles as they like, but can only accept one profile for a job offer. Accepting a profile automatically refuses the other applicants and closes the job offer.

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
