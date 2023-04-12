#### Запрос на сервер

weekPrice.ts
```typescript
export const getWeekPrice = (
  params: string
): AxiosPromise<string> => {
  return Apis.tickets_api.get(
    endpoints_tickets.get_week_price(params)
  )
}
```

endpoints.ts
```typescript
get_week_price: (state: string) => `/week-price/${state}`,
```
#### Форматирование даты

dateTransform.ts
```typescript
export function formatWithFullMonthAndWeekday(
  date: string | Date | number,
  locale: string
) {
  return format(new Date(date), 'dd MMMM, EEEEEE', {
    locale: locales[locale?.toLowerCase() as keyof typeof locales],
  })
}
```
#### Включение компонента в страницу

TicketSearchPage.tsx
```jsx
{aviaTickets?.length ? (
  <Prices
    searchCode={searchCode}
    activeType={activeTab}
    currency={Currency.RUB}
  />
) : null}
```

#### Обработка полученных данных и их вёрстка в TSX, разбито на два компонента.

Prices.tsx
```jsx
interface PricesProps {
  searchCode: string | null
  activeType: number
  currency: Currency
}

export const Prices: FC<PricesProps> = ({
  searchCode,
  activeType,
  currency,
}) => {
  const [pricesContent, setPricesContent] = useState<any>()
  const [activeTab, setActiveTab] = useState<number>(0)

  const handleClick = (index: number) => {
    setActiveTab(index)
  }

  const { t } = useTranslation(['common'])
  let tab = ''

  switch (activeType) {
    case 0:
      tab = t('common:words.airplanes')
      break
    case 1:
      tab = t('common:words.trains')
      break
    case 2:
      tab = t('common:words.buses')
      break
    default:
      tab = 'Неправильное состояние'
  }

  useEffect(() => {
    if (searchCode) getContent(searchCode)
  }, [searchCode])

  const getContent = async (searchCode: string) => {
    const pricesContent = await getWeekPrice(searchCode)
    setPricesContent(pricesContent.data)
  }

  return (
    <div className={s.prices}>
      <div className={s.titleBlock}>
        <div className={s.topText}>Цены на неделю</div>
        <div className={s.bottomText}>{tab.toLowerCase()}</div>
      </div>
      <div className={s.blockContainer}>
        {pricesContent
          ?.slice(0, 7)
          .map((item: any, index: number) => (
            <PriceCard
              key={index}
              item={item}
              isActive={index === activeTab}
              onClick={() => handleClick(index)}
              currency={currency}
            />
          ))}
      </div>
    </div>
  )
}
```

PriceCard.tsx
```jsx
interface PriceCardProps {
  item: any
  currency: Currency
  isActive?: boolean
  onClick?: () => void
}

export const PriceCard: FC<PriceCardProps> = ({
  item,
  currency,
  isActive,
  onClick,
}) => {
  const { t } = useTranslation(['common'])
  const { locale } = useRouter()

  return (
    <div
      className={cn(s.priceBlock, {
        [s.active]: isActive,
      })}
      onClick={onClick}
    >
      <p
        className={cn(s.topText, {
          [s.active]: isActive,
        })}
      >
        {t('common:words.fromLower')} {item.price}{' '}
        {getCurrency(currency)}
      </p>
      <p
        className={cn(s.bottomText, {
          [s.active]: isActive,
        })}
      >
        {formatWithFullMonthAndWeekday(
          item.departure_at,
          locale as string
        )
          .split(' ')
          .map(str => capitalize(str))
          .join(' ')}
      </p>
    </div>
  )
}
```

#### Стилизация компонентов

price.module.scss
```scss
.prices {
  width: 170px;
  background-color: $color-white;
}

.blockContainer {
  @include shadow-input;
  overflow: hidden;
}

.titleBlock {
  padding: 12px 22px;
  display: flex;
  flex-wrap: wrap;
  height: 60px;
  margin-bottom: 15px;
  border: 1px solid $color-light-grey;
  border-radius: 4px;
  cursor: default;
}

.topText {
  font-family: $font-Inter;
  font-weight: $fw-bold;
  @include font-size(14px, $lh: 1.3);
  color: $color-light-black;
}

.bottomText {
  font-family: $font-Montserrat;
  font-weight: $fw-medium;
  @include font-size(12px, $lh: 1.3);
  color: $color-medium-grey;
}
```

priceCard.module.scss
```scss
.priceBlock {
  padding: 12px 22px;
  display: flex;
  flex-wrap: wrap;
  height: 60px;
  border: 1px solid $color-light-grey;
  cursor: pointer;
  &.active {
    background-color: $color-space-blue;
  }
}

.topText {
  font-family: $font-Inter;
  font-weight: $fw-bold;
  @include font-size(14px, $lh: 1.3);
  color: $color-light-black;
  &.active {
    color: $color-white;
  }
}

.bottomText {
  font-family: $font-Montserrat;
  font-weight: $fw-medium;
  @include font-size(12px, $lh: 1.3);
  color: $color-medium-grey;
  &.active {
    color: $color-bright-blue;
  }
}
```
