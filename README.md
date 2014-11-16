# pypact

*Warning: nothing currently implemented!*

pypact is a consumer driven contract testing library that allows mocking of
responses in the consumer codebase, and verification of the interaction in the
provider codebase.

It is an implementation of https://github.com/bethesque/pact-specification.



## Proposed Usage

### 1. Start with your model

```python
class Something(object):

    def __init__(self, name):
        self.name = name

    def __eq__(self, other):
        return isinstance(other, self.__class__) and self.name == other.name
```

### 2. Create a skeleton client class

```python
class MyServiceProviderClient(object):

    def __init__(self, base_uri):
        self.base_uri = base_uri

    def get_something(self):
        # Yet to be implemented because we're doing Test First Development...
        pass
```

# 3. Configure the mock server

```python
@pytest.fixture(scope="session")
def my_consumer(request):
    consumer = pypact.Consumer("My Service Consumer")


@pytest.fixture(scope="session")
def my_service_provider(consumer):
    return pypact.Service("My Service Provider")


@pytest.fixture
def mock_service(request, my_consumer, my_service_provider):
    return my_consumer.has_pact_with(my_service_provider)
```

# 4. Write a failing spec for the client

```python
@pytest.fixture
def subject(my_service_provider):
    subject = MyServiceProviderClient(my_service_provider.base_uri)
    return subject


def test_returns_something(subject, mock_service):
    """
    Subject returns a Something
    """
    (mock_service
        .given("something exists")
        .upon_receiving("a request for something")
        .with_request(method="get", path="/something")
        .will_respond_with(
            status=200,
            headers={'Content=Type': 'application/json'},
            body={'name': 'A small something'}))

    with mock_service:
        something = subject.get_something()

    assert something == Something(name='A small something')
```