# Design Patterns — Поведенческие

## Описание

Поведенческие паттерны отвечают за взаимодействие объектов.

## Strategy

```python
from abc import ABC, abstractmethod

class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float) -> None: ...

class CreditCardPayment(PaymentStrategy):
    def __init__(self, card_number: str):
        self.card_number = card_number

    def pay(self, amount: float) -> None:
        print(f"Paid ${amount} with card {self.card_number}")

class PayPalPayment(PaymentStrategy):
    def __init__(self, email: str):
        self.email = email

    def pay(self, amount: float) -> None:
        print(f"Paid ${amount} via PayPal ({self.email})")

class ShoppingCart:
    def __init__(self, payment: PaymentStrategy):
        self.payment = payment

    def checkout(self, amount: float) -> None:
        self.payment.pay(amount)

# Использование
cart = ShoppingCart(CreditCardPayment("1234-5678"))
cart.checkout(100)

cart.payment = PayPalPayment("alice@example.com")
cart.checkout(50)
```

## Observer

```python
class EventManager:
    def __init__(self):
        self._listeners = {}

    def subscribe(self, event_type: str, callback):
        self._listeners.setdefault(event_type, []).append(callback)

    def notify(self, event_type: str, *args):
        for callback in self._listeners.get(event_type, []):
            callback(*args)

# Использование
events = EventManager()
events.subscribe("user_created", lambda user: print(f"Welcome {user.name}!"))
events.subscribe("user_created", lambda user: send_email(user.email))
events.notify("user_created", user)
```

## Command

```python
from abc import ABC, abstractmethod

class Command(ABC):
    @abstractmethod
    def execute(self) -> None: ...

class CreateOrderCommand(Command):
    def __init__(self, order: Order):
        self.order = order

    def execute(self) -> None:
        self.order.save()
        self.order.send_confirmation()

class CommandHandler:
    def __init__(self):
        self._commands = {}

    def register(self, name: str, command: Command):
        self._commands[name] = command

    def handle(self, name: str):
        self._commands[name].execute()

# Использование
handler = CommandHandler()
handler.register("create_order", CreateOrderCommand(order))
handler.handle("create_order")
```

## Best Practices

✅ **Используйте** Strategy для замены алгоритмов
✅ **Используйте** Observer для слабосвязанных систем
✅ **Используйте** Command для очереди задач

## Ссылки

- [Refactoring Guru — Behavioral Patterns](https://refactoring.guru/design-patterns/behavioral-patterns)
