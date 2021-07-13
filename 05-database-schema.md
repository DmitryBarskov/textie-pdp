# Database schema

```ruby
course: commentable, ratable
    title: text
    description: text
    ---
    belongs_to compoany
    has_many authors, lessons

lesson: ratable, commentable
    order: text
    title: text
    description: text
    ---
    has_many authors, steps

step
    title: text
    content: text
    ---
    has_one exercise

exercise
    type
    ---
    belongs_to step

select_question: exercise
    type = "select_question"
    options: array
    corret_answer: text
    ---

multiple_choice_question: exercise
    type = "multiple_choice_question"
    options: array
    corret_answer: text
    ---

open_question: exercise
    type = "open_question"
    corret_answer: text
    ---

answer
    value: text
    ---
    belongs_to user, exercise

comment: commentable, ratable
    content: text
    ---
    belongs_to commentable, user

user
    bio: text
    name: text
    email: text
    avatar: image
    password: text
    ---
    has_many lessons, blog_posts, comments
    belongs_to company

company
    bio: text
    name: text
    avatar: image
    ---
    has_many users
    belongs_to owner

blog_post: commentable, ratable
    title: text
    content: text
    ---
    belongs_to author, company

rate
    value: number
    ---
    belongs_to user, ratable
```
