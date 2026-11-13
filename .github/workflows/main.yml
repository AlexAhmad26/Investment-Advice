from drafter import *
from bakery import assert_equal


set_website_title("Investment Advice")
set_site_information(
    author="Alex Ahmad",
    description="Help people make good financial decisions when investing",
    sources="Thanks to the Drafter docs and my friends.",
    planning=["plan.pdf", "design.pdf"],
    links=["https://github.com/Alexahmad26/Investment-Advice"]
)

# ---------------------------------------------------------------------------
# ADVANCED TECHNIQUES USED (not covered in class) AND WHAT THEY DO
#  1. try/except: tries code that might fail (turning text into a number) and
#     runs the except block instead of crashing if it does.
#  2. List unpacking with * : [*a, "x"] puts every item of list a into the new list.
#  3. List comprehension: [s for s in items if ...] builds a filtered list in one line.
#  4. Returning/unpacking tuples: a function returns two values at once, and
#     "a, b = f()" stores them in two variables.
#  5. String methods: .startswith() checks how text begins, .strip() removes
#     extra spaces from the ends.
#  6. f-string formatting: {x:.2f} shows 2 decimals, {x:.0f} shows none.
#  7. Type hints like tuple[str, str] and list[Stock]: they only describe types.
#  8. add_website_css: a Drafter function that sets page colors and fonts.
# ---------------------------------------------------------------------------
from dataclasses import dataclass

set_website_title("Stock Helper")

# Styling (try/except means: if add_website_css isn't available, skip it instead of crashing)
try:
    add_website_css("body", "background: #f2f6fb; font-family: 'Segoe UI', Arial, sans-serif; color: #22303f;")
    add_website_css("h1", "color: #1f3a5f; text-align: center;")
    add_website_css("h2", "color: #1f3a5f; border-bottom: 3px solid #2e7d6b; padding-bottom: 6px;")
    add_website_css("button", "background: #2e7d6b; color: white; border: none; border-radius: 8px; "
                              "padding: 8px 16px; margin: 4px; cursor: pointer; font-size: 15px;")
    add_website_css("input, select", "padding: 6px; border: 1px solid #b8c4d6; border-radius: 6px; margin: 4px 0;")
except Exception:
    pass


def verdict_icon(verdict: str) -> str:
    """Pick an emoji that matches the verdict."""
    if verdict.startswith("Good value"):
        return "✅"
    elif verdict.startswith("Fairly priced"):
        return "🟡"
    elif verdict.startswith("Pricey") or verdict.startswith("Expensive"):
        return "⚠️"
    elif verdict.startswith("Can't") or verdict.startswith("Too risky"):
        return "❌"
    return "⚪"


DIVIDER = "─" * 30


@dataclass
class Stock:
    name: str
    price: float
    low_52wk: float
    high_52wk: float
    market_cap: float  # in billions
    long_term: bool
    verdict: str
    reason: str


@dataclass
class State:
    budget: float
    risk_level: str
    stocks: list[Stock]
    message: str


def get_verdict(state: State, stock: Stock) -> tuple[str, str]:
    """Return (verdict, short reason) for this stock and this user."""
    if stock.price > state.budget:
        return ("Can't afford",
                f"The price (${stock.price:.2f}) is above your ${state.budget:.2f} budget.")

    if state.risk_level == "Low" and stock.market_cap < 2:
        return ("Too risky for you",
                f"Its market cap is only ${stock.market_cap}B, which is small for a low-risk investor.")

    shares = int(state.budget // stock.price)
    if shares == 1:
        shares_text = "1 share"
    else:
        shares_text = f"{shares} shares"

    # Where the price sits in the 52-week range (0 = at the low, 1 = at the high)
    span = stock.high_52wk - stock.low_52wk
    if span > 0:
        position = (stock.price - stock.low_52wk) / span
    else:
        position = 0.5
    range_text = f"(${stock.low_52wk:.2f} - ${stock.high_52wk:.2f})"

    buyable = False
    if position <= 1 / 3:
        label = "Good value"
        reason = f"It's priced in the bottom third of its 52-week range {range_text}."
        buyable = True
    elif position >= 2 / 3:
        if stock.long_term:
            label = "Pricey now, but okay for long-term"
            reason = (f"It's in the top third of its 52-week range {range_text}, "
                      "but you plan to hold it for a year or more.")
        else:
            label = "Expensive right now"
            reason = f"It's priced in the top third of its 52-week range {range_text}."
    else:
        label = "Fairly priced"
        reason = f"It's priced in the middle of its 52-week range {range_text}."
        buyable = True

    verdict = label
    if buyable:
        verdict += f" - you could buy {shares_text}"

    # Budget checks: how big a piece of the budget is one share?
    share_of_budget = stock.price / state.budget
    if state.risk_level == "Low":
        max_share = 0.25
    else:
        max_share = 0.5

    if share_of_budget > max_share:
        verdict += " (but a big chunk of your budget)"
        reason += (f" One share uses {share_of_budget * 100:.0f}% of your budget, "
                   f"more than the {max_share * 100:.0f}% suggested for {state.risk_level.lower()} risk.")

    if shares <= 2:
        reason += f" You could only buy {shares_text}, which leaves little room to diversify."

    return (verdict, reason)


def run_analysis(state: State, stock: Stock) -> None:
    # Tuple unpacking (advanced technique #4): get_verdict returns two values at once
    stock.verdict, stock.reason = get_verdict(state, stock)


def message_line(state: State) -> list:
    # Returns a list so other pages can unpack it with * (advanced technique #2)
    if state.message != "":
        return [state.message]
    return []


@route
def index(state: State) -> Page:
    """Show the home page with a short explanation of the site and links to start or view the watchlist."""
    return Page(state, [
        Header("📈 Stock Helper", 1),
        "Not sure if a stock is worth buying? Tell us your budget and the stocks "
        "you're considering, and we'll give each one a verdict.",
        BulletedList([
            "💰 Set your budget and risk level",
            "➕ Add the stocks you're thinking about",
            "🔍 Get a verdict and a reason for each one",
        ]),
        Button("🚀 Get started", setup_page),
        Button("📋 View my watchlist", watchlist_page),
    ])


@route
def setup_page(state: State) -> Page:
    """Show the form where the user enters (or changes) their budget and risk level."""
    current = ""
    if state.budget > 0:
        current = str(state.budget)
    return Page(state, [
        Header("💰 Setup", 2),
        *message_line(state),  # * unpacking (advanced technique #2)
        "How much do you want to invest (in dollars)?",
        TextBox("budget", current),
        "How much risk are you comfortable with?",
        SelectBox("risk_level", ["Low", "Medium", "High"], state.risk_level),
        Button("💾 Save setup", save_setup),
        Button("🏠 Back home", index),
    ])


@route
def save_setup(state: State, budget: str, risk_level: str) -> Page:
    # try/except (advanced technique #1): float() crashes on text like 'abc'
    try:
        amount = float(budget)
    except ValueError:
        state.message = "Budget must be a number."
        return setup_page(state)

    if amount <= 0:
        state.message = "Budget must be greater than 0."
        return setup_page(state)

    state.budget = amount
    state.risk_level = risk_level
    state.message = ""

    # Re-check any stocks that were already analyzed against the new budget
    for stock in state.stocks:
        if stock.verdict != "Not analyzed yet":
            run_analysis(state, stock)

    if len(state.stocks) > 0:
        return watchlist_page(state)
    return add_stock_page(state)


@route
def add_stock_page(state: State) -> Page:
    """Show the form for entering a new stock's name, price, 52-week range, market cap, and hold plan."""
    return Page(state, [
        Header("➕ Add a stock", 2),
        *message_line(state),
        f"💵 Budget: ${state.budget:.2f}  |  🎯 Risk: {state.risk_level}",
        "Stock name or ticker:",
        TextBox("name", ""),
        "Current price ($):",
        TextBox("price", ""),
        "52-week low ($):",
        TextBox("low", ""),
        "52-week high ($):",
        TextBox("high", ""),
        "Market cap (in billions of $):",
        TextBox("market_cap", ""),
        CheckBox("long_term", False),
        "I plan to hold this stock for 1+ year",
        Button("💾 Save stock", save_stock),
        Button("💰 Change budget", setup_page),
        Button("📋 Back to watchlist", watchlist_page),
    ])


@route
def save_stock(state: State, name: str, price: str, low: str, high: str,
               market_cap: str, long_term: bool) -> Page:
    if name.strip() == "":
        state.message = "Please enter a stock name."
        return add_stock_page(state)

    # try/except (advanced technique #1): float() crashes on text like 'abc'
    try:
        price_num = float(price)
        low_num = float(low)
        high_num = float(high)
        cap_num = float(market_cap)
    except ValueError:
        state.message = "Price, 52-week low/high, and market cap must be numbers."
        return add_stock_page(state)

    if price_num <= 0 or low_num > high_num:
        state.message = "Price must be above 0 and the 52-week low can't exceed the high."
        return add_stock_page(state)

    state.stocks.append(
        Stock(name.strip(), price_num, low_num, high_num, cap_num,
              long_term, "Not analyzed yet", "")
    )
    state.message = ""
    return watchlist_page(state)


@route
def watchlist_page(state: State) -> Page:
    """Show every stock with its verdict and reason, plus buttons to analyze, remove, or add stocks."""
    content = [Header("📋 Watchlist", 2), *message_line(state)]
    content.append(f"💵 Budget: ${state.budget:.2f}  |  🎯 Risk: {state.risk_level}")

    if len(state.stocks) == 0:
        content.append("You haven't added any stocks yet.")
        content.append(Button("➕ Add a stock", add_stock_page))
        content.append(Button("💰 Change budget", setup_page))
        content.append(Button("🏠 Back home", index))
        return Page(state, content)

    names = []
    for stock in state.stocks:
        content.append(DIVIDER)
        content.append(f"📊 {stock.name} - ${stock.price:.2f}")
        content.append(f"{verdict_icon(stock.verdict)} {stock.verdict}")
        if stock.reason != "":
            content.append("💬 Why: " + stock.reason)
        names.append(stock.name)

    content.append(DIVIDER)

    content.append(Button("🔍 Analyze all", analyze_all))
    content.append("Or pick one stock:")
    content.append(SelectBox("stock_name", names, names[0]))
    content.append(Button("🔍 Analyze", analyze_stock))
    content.append(Button("🗑️ Remove", remove_stock))
    content.append(Button("➕ Add another stock", add_stock_page))
    content.append(Button("🏆 See results", results_page))
    content.append(Button("💰 Change budget", setup_page))
    return Page(state, content)


@route
def analyze_all(state: State) -> Page:
    for stock in state.stocks:
        run_analysis(state, stock)
    state.message = ""
    return watchlist_page(state)


@route
def analyze_stock(state: State, stock_name: str) -> Page:
    for stock in state.stocks:
        if stock.name == stock_name:
            run_analysis(state, stock)
    state.message = ""
    return watchlist_page(state)


@route
def remove_stock(state: State, stock_name: str) -> Page:
    # List comprehension (advanced technique #3): keep every stock except the removed one
    state.stocks = [s for s in state.stocks if s.name != stock_name]
    state.message = ""
    return watchlist_page(state)


@route
def results_page(state: State) -> Page:
    """Re-analyze all stocks and show the ones that look like good buys for the user's budget."""
    content = [Header("🏆 Results", 2)]
    content.append(f"💵 Budget: ${state.budget:.2f}  |  🎯 Risk: {state.risk_level}")
    good_picks = []

    for stock in state.stocks:
        run_analysis(state, stock)
        if stock.verdict.startswith("Good value"):
            good_picks.append(stock)

    if len(good_picks) > 0:
        content.append("✅ Stocks that look worth buying right now:")
        for stock in good_picks:
            content.append(DIVIDER)
            content.append(f"⭐ {stock.name} - ${stock.price:.2f}")
            content.append(f"💬 {stock.reason}")
    else:
        content.append("🤔 None of your stocks look like a great buy right now.")

    content.append(Button("💰 Change budget", setup_page))
    content.append(Button("📋 Back to watchlist", watchlist_page))
    content.append(Button("🏠 Back home", index))
    return Page(state, content)


# ---------------------------------------------------------------------------
# Unit tests (one test per case)
# ---------------------------------------------------------------------------

def make_stock(name, price, low, high, cap, long_term):
    """Build an unanalyzed Stock for use in the tests below."""
    return Stock(name, price, low, high, cap, long_term, "Not analyzed yet", "")


def run_tests() -> None:
    """Run every unit test. Kept inside a function so the test variables stay local
    instead of becoming global variables."""
    # get_verdict
    assert_equal(
        get_verdict(State(100.0, "Medium", [], ""), make_stock("A", 150.0, 100.0, 200.0, 10.0, False)),
        ("Can't afford", "The price ($150.00) is above your $100.00 budget."))

    assert_equal(
        get_verdict(State(1000.0, "Low", [], ""), make_stock("A", 10.0, 5.0, 20.0, 1.5, False)),
        ("Too risky for you", "Its market cap is only $1.5B, which is small for a low-risk investor."))

    assert_equal(
        get_verdict(State(1000.0, "Medium", [], ""), make_stock("A", 10.0, 10.0, 40.0, 10.0, False)),
        ("Good value - you could buy 100 shares",
         "It's priced in the bottom third of its 52-week range ($10.00 - $40.00)."))

    assert_equal(
        get_verdict(State(1000.0, "Medium", [], ""), make_stock("A", 25.0, 10.0, 40.0, 10.0, False)),
        ("Fairly priced - you could buy 40 shares",
         "It's priced in the middle of its 52-week range ($10.00 - $40.00)."))

    assert_equal(
        get_verdict(State(1000.0, "Medium", [], ""), make_stock("A", 38.0, 10.0, 40.0, 10.0, False)),
        ("Expensive right now",
         "It's priced in the top third of its 52-week range ($10.00 - $40.00)."))

    assert_equal(
        get_verdict(State(1000.0, "Medium", [], ""), make_stock("A", 38.0, 10.0, 40.0, 10.0, True)),
        ("Pricey now, but okay for long-term",
         "It's in the top third of its 52-week range ($10.00 - $40.00), "
         "but you plan to hold it for a year or more."))

    assert_equal(
        get_verdict(State(100.0, "Medium", [], ""), make_stock("A", 60.0, 50.0, 200.0, 10.0, False)),
        ("Good value - you could buy 1 share (but a big chunk of your budget)",
         "It's priced in the bottom third of its 52-week range ($50.00 - $200.00). "
         "One share uses 60% of your budget, more than the 50% suggested for medium risk. "
         "You could only buy 1 share, which leaves little room to diversify."))

    assert_equal(
        get_verdict(State(100.0, "Medium", [], ""), make_stock("A", 40.0, 30.0, 120.0, 10.0, False)),
        ("Good value - you could buy 2 shares",
         "It's priced in the bottom third of its 52-week range ($30.00 - $120.00). "
         "You could only buy 2 shares, which leaves little room to diversify."))

    assert_equal(
        get_verdict(State(100.0, "Low", [], ""), make_stock("A", 30.0, 20.0, 100.0, 5.0, False)),
        ("Good value - you could buy 3 shares (but a big chunk of your budget)",
         "It's priced in the bottom third of its 52-week range ($20.00 - $100.00). "
         "One share uses 30% of your budget, more than the 25% suggested for low risk."))

    assert_equal(
        get_verdict(State(1000.0, "Medium", [], ""), make_stock("A", 10.0, 10.0, 10.0, 10.0, False)),
        ("Fairly priced - you could buy 100 shares",
         "It's priced in the middle of its 52-week range ($10.00 - $10.00)."))

    # run_analysis
    analysis_stock = make_stock("A", 150.0, 100.0, 200.0, 10.0, False)
    run_analysis(State(100.0, "Medium", [], ""), analysis_stock)
    assert_equal(analysis_stock.verdict, "Can't afford")
    assert_equal(analysis_stock.reason, "The price ($150.00) is above your $100.00 budget.")

    # verdict_icon
    assert_equal(verdict_icon("Good value - you could buy 3 shares"), "✅")
    assert_equal(verdict_icon("Fairly priced - you could buy 3 shares"), "🟡")
    assert_equal(verdict_icon("Expensive right now"), "⚠️")
    assert_equal(verdict_icon("Can't afford"), "❌")
    assert_equal(verdict_icon("Not analyzed yet"), "⚪")

    # message_line
    assert_equal(message_line(State(0.0, "Medium", [], "")), [])
    assert_equal(message_line(State(0.0, "Medium", [], "Oops")), ["Oops"])

    # save_setup
    state = State(0.0, "Medium", [], "")
    save_setup(state, "abc", "Low")
    assert_equal(state.message, "Budget must be a number.")

    state = State(0.0, "Medium", [], "")
    save_setup(state, "-5", "Low")
    assert_equal(state.message, "Budget must be greater than 0.")

    state = State(0.0, "Medium", [], "")
    save_setup(state, "500", "High")
    assert_equal((state.budget, state.risk_level), (500.0, "High"))

    # save_stock
    state = State(100.0, "Medium", [], "")
    save_stock(state, "  ", "10", "5", "20", "3", False)
    assert_equal(state.message, "Please enter a stock name.")

    state = State(100.0, "Medium", [], "")
    save_stock(state, "AAPL", "ten", "5", "20", "3", False)
    assert_equal(state.message, "Price, 52-week low/high, and market cap must be numbers.")

    state = State(100.0, "Medium", [], "")
    save_stock(state, "AAPL", "10", "30", "20", "3", False)
    assert_equal(state.message, "Price must be above 0 and the 52-week low can't exceed the high.")

    state = State(100.0, "Medium", [], "")
    save_stock(state, " AAPL ", "10", "5", "20", "3", True)
    assert_equal(state.stocks, [Stock("AAPL", 10.0, 5.0, 20.0, 3.0, True, "Not analyzed yet", "")])

    # analyze_stock
    state = State(100.0, "Medium", [make_stock("A", 150.0, 100.0, 200.0, 10.0, False),
                                     make_stock("B", 150.0, 100.0, 200.0, 10.0, False)], "")
    analyze_stock(state, "A")
    assert_equal([s.verdict for s in state.stocks], ["Can't afford", "Not analyzed yet"])

    # analyze_all
    state = State(100.0, "Medium", [make_stock("A", 150.0, 100.0, 200.0, 10.0, False),
                                     make_stock("B", 150.0, 100.0, 200.0, 10.0, False)], "")
    analyze_all(state)
    assert_equal([s.verdict for s in state.stocks], ["Can't afford", "Can't afford"])

    # remove_stock
    state = State(100.0, "Medium", [make_stock("A", 10.0, 5.0, 20.0, 10.0, False),
                                     make_stock("B", 10.0, 5.0, 20.0, 10.0, False)], "")
    remove_stock(state, "A")
    assert_equal([s.name for s in state.stocks], ["B"])


run_tests()


start_server(State(0.0, "Medium", [], ""))
