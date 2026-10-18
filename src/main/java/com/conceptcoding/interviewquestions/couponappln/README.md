# Coupon Application System — Low Level Design

Shopping cart that applies discount coupons to products. Coupons are stacked on top of products using the **Decorator** pattern, so the cart total reflects every applicable discount.

**Package:** `com.conceptcoding.interviewquestions.couponappln`

## Entry Point

`Demo.main()` — creates four products (Fan, Office Chair, Omega3-Tabs, Shirt) across different `ProductType`s, adds them to a `ShoppingCart`, and prints each applied coupon and the total price after discount.

Run `Demo` directly from your IDE.

## Folder Structure

```
couponappln/
├── Demo.java                         # Entry point (main method)
├── ShoppingCart.java                 # Adds products wrapped in coupons, computes total
├── coupons/
│   ├── CouponDecorator.java          # Abstract decorator wrapping a Product
│   ├── PercentageCouponDecorator.java# Flat percentage discount on any product
│   └── TypeCouponDecorator.java      # Percentage discount for eligible product types
└── product/
    ├── Product.java                  # Abstract product: name, original price, type
    ├── Item1.java                    # Concrete products
    ├── Item2.java
    ├── Item3.java
    ├── Item4.java
    └── ProductType.java              # FURNITURE, ELECTRONICS, CLOTHES, PHARMACY
```

## Classes

### Core

| Class | Responsibility |
|-------|----------------|
| `Demo` | Entry point; builds products and a cart, prints the discounted total. |
| `ShoppingCart` | `addToCart()` wraps each product as `TypeCouponDecorator(PercentageCouponDecorator(product, 20), 10)` before storing it. `getTotalPrice()` sums `getPrice()` of the decorated products. |

### Coupons (`coupons/`)

| Class | Responsibility |
|-------|----------------|
| `CouponDecorator` | Abstract class that **is a** `Product` and **has a** `Product`, plus a `discountPercentage`. |
| `PercentageCouponDecorator` | Applies a percentage discount to the wrapped product's price. |
| `TypeCouponDecorator` | Applies a percentage discount only when the product type is `FURNITURE` or `ELECTRONICS`; otherwise returns the price unchanged. |

### Products (`product/`)

| Class | Responsibility |
|-------|----------------|
| `Product` | Abstract base with `name`, `originalPrice`, `type` and abstract `getPrice()`. |
| `Item1` – `Item4` | Concrete products; `getPrice()` returns the original price. |
| `ProductType` | Enum: `FURNITURE`, `ELECTRONICS`, `CLOTHES`, `PHARMACY`. |

## Design Notes

- **Decorator pattern:** each coupon wraps a `Product` and adjusts `getPrice()`, so coupons can be stacked in any order without changing product classes.
- **Extensible:** new coupon types only need a new `CouponDecorator` subclass.
