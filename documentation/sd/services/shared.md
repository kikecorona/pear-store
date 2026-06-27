# shared

> *Auto-generated SD page.* Service `shared` · revision `50b25eca0421` · content hash `50b25eca0421` · last updated 2026-06-27 05:55 UTC.

## Endpoints

_None detected._

## Data structures

- **`App`** (dataclass, `models.py`) — `id: str`, `name: str`, `developer: str`, `category: str`, `price_cents: int`, `description: str`, `icon_emoji: str`, `rating: float`, `rating_count: int`, `bundle_id: str`, `version: str`, `size_mb: int`
- **`User`** (dataclass, `models.py`) — `id: str`, `email: str`, `display_name: str`, `password: str`, `payment_method_token: Optional[str]`
- **`CartItem`** (dataclass, `models.py`) — `app_id: str`, `added_at: str`
- **`Order`** (dataclass, `models.py`) — `id: str`, `user_id: str`, `items: list`, `total_cents: int`, `status: str`, `payment_id: Optional[str]`, `created_at: str`

## Downstream dependencies

_No downstream dependencies detected._


## Related products

_No B&P pages currently reference this service._
