## Iterating Through a Paginated API

With growing amounts of data, every API deploys some form of pagination. A myriad of articles and books about API design patterns can teach us how to set up different kinds of pagination; however, what can we do to make our lives easier when consuming such an API?

In this article, I dive into my preferred approach for working with paginated APIs. While the code presented is in PHP, the same approach can be employed with any programming language that has the notion of an [iterator](https://en.wikipedia.org/wiki/Iterator).

The examples shown are based on working with an API employing _page-based pagination_, but with slight tweaks, it would work with _cursor-based_ or _keyset-based_ pagination as well.

---

### The Imperative Way

#### The API

Let's assume we're working with [Modulr](https://www.modulrfinance.com/), a platform and API that allows businesses to make and receive payments. Their API allows us to [retrieve payments](https://modulr.readme.io/reference/getpayments) via a `GET /payments` endpoint call.

Modulr's API has consistent response bodies when it comes to fetching data, which we can represent as an interface:

```PHP
/**
 * @template T
 */
interface PaginatedResultsInterface
{
  /** @return list<T> */
  public function getContent(): array;

  public function getSize(): int;

  public function getTotalSize(): int;

  public function getPage(): int;

  public function getTotalPages(): int;
}
```

#### The Initial Approach

We might want to retrieve data from an API and do some further processing with it; in our case, say we need to match payments from the Modulr API with transactions recorded in our database, one day at a time.

Assuming we've already set up a client to call the API and deserialize the response nicely into objects satisfying our interface, a typical approach for this would look as follows:

```PHP
class PaymentsService
{
  public function __construct(private ModulrClient $modulrClient) {}

  public function matchPayments(DateTimeInterface $from, DateTimeInterface $to): void
  {
    $currentPage = 0;

    do {
      $results = $this->modulrClient->getPayments($from, $to, $currentPage);

      $this->process($results);

      $currentPage++;
    } while ($currentPage < $results->getTotalPages());
  }

  private function process(PaginatedResultsInterface $results): void
  {
    // here we run DB queries, match payments, check amounts, etc.
  }
}
```

### The Iterative Way

Our initial approach surely does its job, but presents a few elements that are less than ideal:

- mixing business logic with lower-level boilerplate code
- increased testing demand if we want to cover all edge cases
- potential for cyclomatic complexity to greatly increase if more logic is needed in the future

Of course, we could move some of the logic for paginating through the API's results to a separate class. That's exactly what we'll do by implementing a PHP [iterator](https://www.php.net/manual/en/class.iterator.php).

#### Enter: `PaginatingIterator`

To implement the `Iterator` interface we need to define the logic for its methods, which will be called in this specific order when iterating via a `foreach` loop:

- `rewind` - first time only
- `valid -> current -> key -> next` - repeat as long as `valid` returns _true_

The complete code for the `PaginatingIterator` is available as a [Gist here](https://gist.github.com/AlexandruGG/5f482a7a2ef4fab9f2890f4a17e96a9f); main aspects to note are:

- we don't want to call the API until we start iterating
- at each step of iteration we yield the current page of results' content
- we keep going until we reach the last page or we receive an empty result set
- if we don't want to start from the _first_ page we can specify the initial page to fetch and the iterator will remember it

#### The Result

With our shiny new iterator, the _payments domain_ code becomes:

```PHP
class PaymentsService
{
  public function __construct(private ModulrClient $modulrClient) {}

  public function matchPayments(DateTimeInterface $from, DateTimeInterface $to): void
  {
    $paginatingIterator = new PaginatingIterator(
      fn (int $page): PaginatedResultsInterface => $this->modulrClient->getPayments($page, $from, $to)
    );

    foreach ($paginatingIterator as $results) {
      $this->process($results);
    }
  }

  private function process(PaginatedResultsInterface $results): void
  {
    // here we run DB queries, match payments, check amounts, etc.
  }
}
```

You might look at the two approaches and say that the second one resulted in more code overall. And you'd be completely right! For simple cases and single-use instances creating the iterator would be overkill. However, if we're planning to work with multiple endpoints from the same API, the benefits start to add up.

Our "domain code" becomes simpler and easier to test, and we can be confident that the iterator will work the same way in each situation (assuming we've written tests for it, of course).

To reuse our `PaginatingIterator` with other endpoints, all we have to do is instantiate it with the appropriate closure:

```PHP
$customers = new PaginatingIterator(
  fn (int $page): PaginatedResultsInterface => $this->modulrClient->getCustomers($page, ...)
);

$accounts = new PaginatingIterator(
  fn (int $page): PaginatedResultsInterface => $this->modulrClient->getAccounts($page, ...)
);

$beneficiaries = new PaginatingIterator(
  fn (int $page): PaginatedResultsInterface => $this->modulrClient->getBeneficiaries($page, ...)
);
```

#### Please Sir, May I Have Some More?

The world of iterators has much more to offer, and once you dive in you won't want to go back. Check out the iterators provided by the [Standard PHP Library](https://www.php.net/manual/en/spl.iterators.php), as well as those available through [composer packages](https://packagist.org/?query=iterators) such as the excellent [loophp/iterators](https://github.com/loophp/iterators).

#### Happy Iterating!
