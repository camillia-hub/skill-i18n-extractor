# Extraction examples and skip list

## Skip list (leave in code)

- CSS classes, `data-testid`, `id` attributes, route paths, API field names
- Log messages, telemetry events, and exception messages aimed at developers
- Date/number format strings and regex patterns
- Strings already passed to `t(...)`, `FormattedMessage`, or `_localizer[...]`
- Single characters used as separators (`" / "`, `"-"`)

## React (react-i18next)

Before:

```tsx
export function OrderActions({ count }: { count: number }) {
  return (
    <div>
      <button title="Submit the order">Submit order</button>
      <p>{count === 1 ? "1 item in cart" : `${count} items in cart`}</p>
    </div>
  );
}
```

After:

```tsx
import { useTranslation } from "react-i18next";

export function OrderActions({ count }: { count: number }) {
  const { t } = useTranslation();
  return (
    <div>
      <button title={t("orders.actions.submitTooltip")}>
        {t("orders.actions.submitButton")}
      </button>
      <p>{t("orders.actions.itemsInCart", { count })}</p>
    </div>
  );
}
```

Resource file `src/i18n/en.json`:

```json
{
  "orders": {
    "actions": {
      "submitButton": "Submit order",
      "submitTooltip": "Submit the order",
      "itemsInCart_one": "{{count}} item in cart",
      "itemsInCart_other": "{{count}} items in cart"
    }
  }
}
```

Notes:

- Plural suffixes are `_one` / `_other`; add `_zero` when a target locale needs a distinct zero form.
- The interpolation variable must be named `count` for react-i18next plural resolution to kick in.

## React (react-intl / ICU)

```tsx
<FormattedMessage
  id="orders.actions.itemsInCart"
  defaultMessage="{count, plural, one {# item in cart} other {# items in cart}}"
  values={{ count }}
/>
```

## .NET (IStringLocalizer)

Before:

```csharp
public IActionResult Approve(int id)
{
    TempData["Message"] = "Order approved";
    return RedirectToAction("Detail", new { id });
}
```

After:

```csharp
public class OrdersController : Controller
{
    private readonly IStringLocalizer<OrdersController> _localizer;

    public OrdersController(IStringLocalizer<OrdersController> localizer)
    {
        _localizer = localizer;
    }

    public IActionResult Approve(int id)
    {
        TempData["Message"] = _localizer["orders.detail.approvedMessage"];
        return RedirectToAction("Detail", new { id });
    }
}
```

Resource `Resources/Controllers.OrdersController.en.resx` (or the repo's JSON equivalent) with key `orders.detail.approvedMessage`.

Razor view:

```cshtml
@inject IViewLocalizer Localizer
<h1>@Localizer["orders.detail.title"]</h1>
```

Interpolation in .NET: the resource holds `"You have {0} unread messages"` and the call passes the value: `_localizer["inbox.unread", count]`.
