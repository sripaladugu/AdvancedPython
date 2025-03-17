Favor Composition Over Inheritance
Composition is a design principle in object-oriented programming that promotes code reuse by allowing objects to contain other objects. This approach is an alternative to inheritance, where one class inherits from another and extends its functionality. Composition can lead to more flexible and maintainable code by favoring object composition over class inheritance.

Composition over inheritance is a design principle that suggests using composition to achieve code reuse and flexibility rather than relying heavily on inheritance hierarchies. This approach can lead to more maintainable and adaptable code by favoring the combination of simple, independent objects over a complex inheritance structure.

** Key Characteristics of Composition: **

- Combining Objects: Composition involves creating objects that contain other objects as attributes.
- Flexibility: Composition allows you to change the behavior of an object by changing the objects it contains.
- Code Reusability: Composition promotes code reusability by allowing you to reuse objects in different contexts.

** Example: **

Let's start with Employee Management System. We have three types of employees: HourlyEmployee, SalariedEmployee, and Freelancer. Each employee has different attributes and methods, but they all share a common behavior: they can compute how much they should be paid based on their specific rules.

```python
"""
Very advanced Employee management system.
"""

from dataclasses import dataclass


@dataclass
class HourlyEmployee:
    """Employee that's paid based on number of worked hours."""

    name: str
    id: int
    commission: float = 100
    contracts_landed: float = 0
    pay_rate: float = 0
    hours_worked: int = 0
    employer_cost: float = 1000

    def compute_pay(self) -> float:
        """Compute how much the employee should be paid."""
        return (
            self.pay_rate * self.hours_worked
            + self.employer_cost
            + self.commission * self.contracts_landed
        )


@dataclass
class SalariedEmployee:
    """Employee that's paid based on a fixed monthly salary."""

    name: str
    id: int
    commission: float = 100
    contracts_landed: float = 0
    monthly_salary: float = 0
    percentage: float = 1

    def compute_pay(self) -> float:
        """Compute how much the employee should be paid."""
        return (
            self.monthly_salary * self.percentage
            + self.commission * self.contracts_landed
        )


@dataclass
class Freelancer:
    """Freelancer that's paid based on number of worked hours."""

    name: str
    id: int
    commission: float = 100
    contracts_landed: float = 0
    pay_rate: float = 0
    hours_worked: int = 0
    vat_number: str = ""

    def compute_pay(self) -> float:
        """Compute how much the employee should be paid."""
        return (
            self.pay_rate * self.hours_worked + self.commission * self.contracts_landed
        )


def main() -> None:
    """Main function."""

    henry = HourlyEmployee(name="Henry", id=12346, pay_rate=50, hours_worked=100)
    print(
        f"{henry.name} worked for {henry.hours_worked} hours and earned ${henry.compute_pay()}."
    )

    sarah = SalariedEmployee(
        name="Sarah", id=47832, monthly_salary=5000, contracts_landed=10
    )
    print(
        f"{sarah.name} landed {sarah.contracts_landed} contracts and earned ${sarah.compute_pay()}."
    )


if __name__ == "__main__":
    main()
```

** Refactoring with Inheritance: **

```python
"""
Very advanced Employee management system.
"""

from abc import ABC, abstractmethod
from dataclasses import dataclass


@dataclass
class Employee(ABC):
    """Basic representation of an employee at the company."""

    name: str
    id: int

    @abstractmethod
    def compute_pay(self) -> float:
        """Compute how much the employee should be paid."""


@dataclass
class HourlyEmployee(Employee):
    """Employee that's paid based on number of worked hours."""

    pay_rate: float
    hours_worked: int = 0
    employer_cost: float = 1000

    def compute_pay(self) -> float:
        return self.pay_rate * self.hours_worked + self.employer_cost


@dataclass
class SalariedEmployee(Employee):
    """Employee that's paid based on a fixed monthly salary."""

    monthly_salary: float
    percentage: float = 1

    def compute_pay(self) -> float:
        return self.monthly_salary * self.percentage


@dataclass
class Freelancer(Employee):
    """Freelancer that's paid based on number of worked hours."""

    pay_rate: float
    hours_worked: int = 0
    vat_number: str = ""

    def compute_pay(self) -> float:
        return self.pay_rate * self.hours_worked


@dataclass
class SalariedEmployeeWithCommission(SalariedEmployee):
    """Employee that's paid based on a fixed monthly salary and that gets a commission."""

    commission: float = 100
    contracts_landed: float = 0

    def compute_pay(self) -> float:
        return super().compute_pay() + self.commission * self.contracts_landed


@dataclass
class HourlyEmployeeWithCommission(HourlyEmployee):
    """Employee that's paid based on number of worked hours and that gets a commission."""

    commission: float = 100
    contracts_landed: float = 0

    def compute_pay(self) -> float:
        return super().compute_pay() + self.commission * self.contracts_landed


@dataclass
class FreelancerWithCommission(Freelancer):
    """Freelancer that's paid based on number of worked hours and that gets a commission."""

    commission: float = 100
    contracts_landed: float = 0

    def compute_pay(self) -> float:
        return super().compute_pay() + self.commission * self.contracts_landed


def main() -> None:
    """Main function."""

    henry = HourlyEmployee(name="Henry", id=12346, pay_rate=50, hours_worked=100)
    print(
        f"{henry.name} worked for {henry.hours_worked} hours and earned ${henry.compute_pay()}."
    )

    sarah = SalariedEmployeeWithCommission(
        name="Sarah", id=47832, monthly_salary=5000, contracts_landed=10
    )
    print(
        f"{sarah.name} landed {sarah.contracts_landed} contracts and earned ${sarah.compute_pay()}."
    )


if __name__ == "__main__":
    main()
```

** Refactoring with Composition: **

```python
"""
Very advanced Employee management system.
"""

from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional


class Contract(ABC):
    """Represents a contract and a payment process for a particular employeee."""

    @abstractmethod
    def get_payment(self) -> float:
        """Compute how much to pay an employee under this contract."""

class Commission(ABC):
    """Represents a commission payment process."""

    @abstractmethod
    def get_payment(self) -> float:
        """Returns the commission to be paid out."""


@dataclass
class ContractCommission(Commission):
    """Represents a commission payment process based on the number of contracts landed."""

    commission: float = 100
    contracts_landed: int = 0

    def get_payment(self) -> float:
        """Returns the commission to be paid out."""
        return self.commission * self.contracts_landed


@dataclass
class Employee:
    """Basic representation of an employee at the company."""

    name: str
    id: int
    contract: Contract
    commission: Optional[Commission] = None

    def compute_pay(self) -> float:
        """Compute how much the employee should be paid."""
        payout = self.contract.get_payment()
        if self.commission is not None:
            payout += self.commission.get_payment()
        return payout


@dataclass
class HourlyContract(Contract):
    """Contract type for an employee being paid on an hourly basis."""

    pay_rate: float
    hours_worked: int = 0
    employer_cost: float = 1000

    def get_payment(self) -> float:
        return self.pay_rate * self.hours_worked + self.employer_cost


@dataclass
class SalariedContract(Contract):
    """Contract type for an employee being paid a monthly salary."""

    monthly_salary: float
    percentage: float = 1

    def get_payment(self) -> float:
        return self.monthly_salary * self.percentage


@dataclass
class FreelancerContract(Contract):
    """Contract type for a freelancer (paid on an hourly basis)."""

    pay_rate: float
    hours_worked: int = 0
    vat_number: str = ""

    def get_payment(self) -> float:
        return self.pay_rate * self.hours_worked


def main() -> None:
    """Main function."""

    henry_contract = HourlyContract(pay_rate=50, hours_worked=100)
    henry = Employee(name="Henry", id=12346, contract=henry_contract)
    print(
        f"{henry.name} worked for {henry_contract.hours_worked} hours "
        f"and earned ${henry.compute_pay()}."
    )

    sarah_contract = SalariedContract(monthly_salary=5000)
    sarah_commission = ContractCommission(contracts_landed=10)
    sarah = Employee(
        name="Sarah", id=47832, contract=sarah_contract, commission=sarah_commission
    )
    print(
        f"{sarah.name} landed {sarah_commission.contracts_landed} contracts "
        f"and earned ${sarah.compute_pay()}."
    )


if __name__ == "__main__":
    main()
```
