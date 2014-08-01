---
author:   jeremyf
category: practices
filename: 2015-08-01-crafting-an-expressive-view-test.md
layout:   post
tagline:  Making Sure the Test Code Conveys Meaning
title:    Crafting an Expressive View Test
tags:     rails, testing
---

*The following examples are paraphrases of the existing features.*

I was wrapping up a feature for Hydramata::Works to [verify the correctness of Edit form rendering](https://github.com/ndlib/hydramata-works/commit/d44e9cf5).

As I was going over the code with Chris, our Quality Assurance specialist, he helped me unearth an undiscovered bug [in the current state of the spec](https://github.com/ndlib/hydramata-works/blob/02ef8dc0b8f644ca95bd5de518dc6f1f9e5e515d/spec/features/in_memory_to_output_buffer_spec.rb#L62-L66).

For reference, here is the rendered output

```html
<form>
  <input name="work[title][]" value="Hello">
  <input name="work[title][]">
</form>
```

The spec that was passing but not correct.

```ruby
expect(rendered_output).to have_tag('form') do
  with_tag('input', value: 'Hello', with: { name: 'work[title][]' })
  with_tag('input', value: '', with: { name: 'work[title][]' })
end
```

On the surface, it looked like I was testing things.
I had written a failing test, wrote the code, and the test passed.
But it was wrong.

It turns out that the `value: ''` and `value: 'Hello'` options were not being used.
I found this out because Chris insisted that I switch some input values.

And poof! Failure.

With some digging around in the [rpsec-html-matchers gem](https://github.com/kucaahbe/rspec-html-matchers), I came up with the following solution:

```ruby
expect(rendered_output).to have_tag('form') do
  with_tag('input', with: { name: 'work[title][]', value: 'Hello' })
  with_tag('input', with: { name: 'work[title][]' }, without: { value: /.*/ })
end
```

In the case of a value that was present, I moved the parameter into the `with:` option.
But the case of the no value was a pernicious creature.

If I were coming back to that spec in a month, I would wonder what is up with `without: { value: /.*/ }`.
It is a negation and a greedy regular expression.
I started to write a commment and stopped.

I dislike comments in code.
And that goes double for tests.
I find them full of lies.
But that is for another time.

All the while Chris was sitting over my shoulder.
And I was talking through the solution.
I was explaining why the test was correct but unhelpful.

So I decided to change the view to make my code easier to test.

```html
<form>
  <div class="title"
    <div class="values">
      <input name="work[title][]" value="Hello" class="existing-input">
      <input name="work[title][]" class="blank-input">
    </div>
  </div>
</form>
```

The above change helped to reflect my intention (See the [actual code change for more context](https://github.com/ndlib/hydramata-works/commit/d44e9cf5)).

My view was now expressing an intent.
In fact on writing this post, I realize that my intent could be more expressive.
I could better clarify the two input class names.
But that is for a future pull request.

The resulting view spec was more expressive.

```ruby
expect(rendered_output).to have_tag('form') do
  with_tag('input.existing-input', with: { name: 'work[title][]', value: 'Hello' })
  with_tag('input.blank-input', with: { name: 'work[title][]' })
end
```

But is it correct?
Upon reflection.
No.

The spec expressed intent but did not verify the technical purpose.
So I [adjusted the specs](https://github.com/ndlib/hydramata-works/commit/656b40418458dd425002c00f6158962318e827cc).

```ruby
expect(rendered_output).to have_tag('form') do
  with_tag('input.existing-input', with: { name: 'work[title][]', value: 'Hello' })
  with_tag('input.blank-input', with: { name: 'work[title][]' }, without: { value: /.*/ })
end
```

I find it maintains it's expression and verifies the technical aspect.
The nagging obtuse technical detail is relegated to the end of the spec; It is still there but is an implementation detail.
