# Daml "University" Capstone Project
This project, built in Daml, deals with student applications to universities.

### I. Overview 
A *student* can apply to a *university*. The *university* asks some *teachers*
to review the application and decides whether to welcome or reject the student
based on a score determined by the gathered reviews.

### II. Workflow
1. *student* creates an `ApplicationRequest` contract mentioning the targeted
   *university* and containing a cover letter that fits length constraints
2. *university* exercises `ProcessRequest` and decides who will be the
   reviewing *teachers* and how many reviews are required at least. This
   creates an `ApplicationPending` contract, and one `ReviewRequest` contract
   for each of the *teachers*.
3. *teacher* exercises `Evaluate` or `Score` on their `ReviewRequest` contract
   to create a corresponding `Review` contract.
4. *student* may exercise the non consuming choice `Consult` on any `Review` to
   check the score they were given.
5. *university* queries for `Review` contracts offchain and exercises
   `GatherReviews` on the desired reviews. This in turn exercises `Fetch` on
   the targetted `Review` contracts. If enough reviews have been collected,
   this creates a `Rejection` or `Offer` contract. If not, this creates an
   updated `ApplicationPending` contract.

### III. Challenge(s)

* In the initial design, the `GatherReviews` choice performed a lookup for all
  the `Review` contracts onchain, based on their `(university, student,
  teacher)` key, iterating on the *teachers*. I realised that it was possible
  for an observer to fetch, but not lookup by key. `Review` contracts would
  have needed the university as a key maintainer, and in turn as a signatory,
  while creating a review is supposed to be done from the teacher's
  perspective. From there, I saw two possible design modifications:
    1. Have a choice `AddReview` in `ApplicationPending` that the teachers can
       exercise. Since there is a whole set of teachers who should be able to
       do that, I would need to have the controller as a parameter of
       `AddReview` and check with an assertion that the controller is indeed
       part of the teachers mentioned in the current payload. This is what I
       would have done in a real case.
    2. **Current choice:** Instead of an onchain lookup, perform an offchain
       query of the reviews and pass their contract ids as a parameter of
       `GatherReviews`. I chose this alternative because it required little
       alteration and made it possible to showcase more onchain and offchain
       iterations for the sake of this capstone project. This introduces a few
       shortcomings in the business logic:
        * The university can use selection bias and only cherry pick the
          reviews they like.
        * In the original design, when enough reviews were collected, remaining
          review requests where automatically cancelled. Now, they are left to
          be dealt with manually.

### IV. Compiling & Testing

To run unit tests and example scenarios, run the pre-written scripts in
`/daml/Test.daml`.

To compile and run Navigator:

```
$ daml start
```
