## Load Testing an API with Artillery

Every API has its limitations when it comes to performance, whether they are visible or not. While monitoring tools help us ensure that everything behaves as expected on a day-to-day basis, sometimes it's necessary to demonstrate that good functionality is maintained at higher loads.

Load testing is a useful tool that can surface an API's limits or provide peace of mind to both internal developers and consumers. Here, I showcase my recent experience with this process using a modern open-source tool called [Artillery](https://www.artillery.io).

---

### Intro + Context

#### Load Testing

Load testing is a type of performance testing undertaken to understand the behaviour of a system under a specific expected load. For example, this can come in the form of an expected concurrent number of users performing a certain operation using an application within a set duration.

In the context of an API, load testing can show how many concurrent requests a given endpoint can serve before performance starts to take a hit. Going even further, we can test a certain flow that we know users would typically follow on a given page by accessing multiple endpoints.

#### My Task

My first proper dive into load testing came when a large client of the company I worked for at the time, [SamKnows](https://www.samknows.com), commissioned us to test a newish API of ours they were using. This API is used to run internet connectivity speed tests on an enabled device, and the client intended to significantly increase their usage of it.

The ask was to prove that the API can perform without any penalty under an expected number of concurrent requests per second. In this case, the test would involve calling a single endpoint with a fairly high response time of around 10 seconds - the amount of time it took to run the speed test.

The initial test uncovered multiple issues which required addressing, after which the API was able to perform as expected. The main problems seen were:

- an inefficient database query using an unindexed table column (classic!)
- a low timeout value on an API client that calls another API
- an insufficient number of child processes allowed by the [php-fpm config](https://www.php.net/manual/en/install.fpm.configuration.php) (this was a PHP API)

### Using Artillery

#### Intro

When looking at a tool to use for my load testing task, I decided to go with Artillery for [multiple reasons](https://www.artillery.io/docs/guides/overview/why-artillery): the documentation looked excellent, the tool seemed easy to use, and everything appeared built with developer productivity in mind.

Tests with Artillery are written as YAML files, and they can range from super simple to complex scenarios or user flows. For example:

```YAML
config:
  target: "https://example.com/api"
  phases:
    - duration: 60
      arrivalRate: 5
      name: Warm up
    - duration: 600
      arrivalRate: 50
      name: Sustained load
```

The above is a simplified example from their documentation; it illustrates a scenario with two phases: a "warm-up phase" where 5 concurrent requests are made every second for 60 seconds, and a second "sustained load phase" where 50 requests are made every second for 10 minutes.

Assuming you've installed the CLI and placed the test script in a `test.yaml` file, all you need to do is: `artillery run test.yaml`.

**Tip:** if you want to see both the requests made and the responses received in a log file, you can use the [debugging options](https://www.artillery.io/docs/guides/guides/http-reference#debugging):

```SH
DEBUG=http* artillery run test.yaml test.log
```

#### Extra Features

Artillery has many features that can make it a valuable tool for the right scenarios/teams. Some of the ones I've tried or plan to try in the future:

- providing payload data [from a CSV file](https://www.artillery.io/docs/guides/guides/test-script-reference#payload---data-from-csv)
- [expectations and assertions](https://www.artillery.io/docs/guides/plugins/plugin-expectations-assertions) for HTTP APIs
- [GitHub Actions integration](https://www.artillery.io/docs/guides/integration-guides/github-actions)
- the [Playground](https://www.artillery.io/docs/guides/guides/playground)

#### The Results

A shortened version of the results reported by Artillery will look like below:

```LOG
http.codes.200: .................................................... 100
http.request_rate: ................................................. 10/sec
http.requests: ..................................................... 100
http.response_time:
  min: ............................................................. 274
  max: ............................................................. 381
  median: .......................................................... 295.9
  p95: ............................................................. 333.7
  p99: ............................................................. 340.4
http.responses: .................................................... 100
```

In this instance, we made _10 requests per second_ for 10 seconds, for a total of 100 requests, and received 100 responses. All of them had a _200 OK_ response code, with a median response time of _295.9 ms_. While there was a slight variation in performance, we can confidently say that 99% of requests finished in _340.4 ms_ or less.

#### Happy Load Testing!
