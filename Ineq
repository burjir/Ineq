"""Inequality Arcade, an Algebra 1 number-line studio and practice game."""

from dataclasses import dataclass
from fractions import Fraction
from html import escape
from math import floor
import operator
import random
import re

import streamlit as st


SIGNS = ("<", "≤", ">", "≥")
FLIP = {"<": ">", "≤": "≥", ">": "<", "≥": "≤"}
TEX_SIGNS = {"<": "<", "≤": r"\le", ">": ">", "≥": r"\ge"}
COMPARE = {"<": operator.lt, "≤": operator.le, ">": operator.gt, "≥": operator.ge}
LEVELS = ("Graph basics", "Solve & graph", "Sign flips", "Variables on both sides")
NUMBER_PATTERN = re.compile(r"[+-]?(?:[0-9]+(?:\.[0-9]*)?|\.[0-9]+|[0-9]+\s*/\s*[+-]?[0-9]+)")


@dataclass(frozen=True)
class Solution:
    boundary: Fraction
    sign: str

    @property
    def closed(self):
        return self.sign in ("≤", "≥")

    @property
    def direction(self):
        return "Left" if self.sign in ("<", "≤") else "Right"

    @property
    def latex(self):
        return f"x {TEX_SIGNS[self.sign]} {number_latex(self.boundary)}"


@dataclass(frozen=True)
class Inequality:
    # a*x + b  sign  c*x + d
    a: int
    b: int
    sign: str
    c: int
    d: int

    @property
    def latex(self):
        return inequality_latex(self.a, self.b, self.sign, self.c, self.d)

    @property
    def coefficient(self):
        return self.a - self.c

    @property
    def constant(self):
        return self.d - self.b

    def solve(self):
        """Return a ray, True for all real numbers, or False for no solution."""
        if self.coefficient == 0:
            return COMPARE[self.sign](0, self.constant)
        sign = FLIP[self.sign] if self.coefficient < 0 else self.sign
        return Solution(Fraction(self.constant, self.coefficient), sign)

    def values_at(self, x):
        return self.a * x + self.b, self.c * x + self.d


def number_text(value):
    return str(value).replace("-", "−")


def number_latex(value):
    value = Fraction(value)
    if value.denominator == 1:
        return str(value.numerator)
    minus = "-" if value < 0 else ""
    return rf"{minus}\frac{{{abs(value.numerator)}}}{{{value.denominator}}}"


def read_number(text):
    cleaned = text.strip().replace("−", "-")
    if len(cleaned) > 24 or not NUMBER_PATTERN.fullmatch(cleaned):
        raise ValueError("Enter a number like −3, 0.5, or 3/2.")
    if "/" in cleaned:
        numerator, denominator = cleaned.split("/")
        if int(denominator) == 0:
            raise ValueError("A fraction needs a denominator other than 0.")
        value = Fraction(int(numerator), int(denominator))
    else:
        value = Fraction(cleaned)
    if abs(value) > 9999:
        raise ValueError("Choose a value from −9,999 to 9,999.")
    return value


def expression_latex(coefficient, constant):
    if coefficient == 0:
        return number_latex(constant)
    term = {1: "x", -1: "-x"}.get(coefficient, f"{coefficient}x")
    if constant == 0:
        return term
    join = "+" if constant > 0 else "-"
    return f"{term} {join} {number_latex(abs(constant))}"


def inequality_latex(a, b, sign, c, d):
    return f"{expression_latex(a, b)} {TEX_SIGNS[sign]} {expression_latex(c, d)}"


def operation(value, variable=False):
    verb = "Add" if value > 0 else "Subtract"
    amount = number_text(abs(value))
    if variable:
        amount = "x" if abs(value) == 1 else f"{amount}x"
    return f"{verb} {amount} {'to' if value > 0 else 'from'} both sides."


def solving_steps(problem):
    steps = []
    if problem.c:
        steps.append((
            operation(-problem.c, variable=True),
            inequality_latex(problem.coefficient, problem.b, problem.sign, 0, problem.d),
        ))
    if problem.b:
        steps.append((
            operation(-problem.b),
            inequality_latex(problem.coefficient, 0, problem.sign, 0, problem.constant),
        ))
    result = problem.solve()
    if isinstance(result, bool):
        if problem.a == problem.c == 0:
            explanation = "The inequality is true for every x." if result else "The inequality is false for every x."
        else:
            explanation = (
                "The variable cancels. The remaining statement is true, so every real number works."
                if result else
                "The variable cancels. The remaining statement is false, so no real number works."
            )
        steps.append((explanation, None))
    elif problem.coefficient != 1:
        verb = "reverse" if problem.coefficient < 0 else "keep"
        steps.append((
            f"Divide both sides by {number_text(problem.coefficient)} and {verb} the inequality sign.",
            result.latex,
        ))
    return steps


def circle_explanation(solution):
    boundary = number_text(solution.boundary)
    if solution.closed:
        return f"Use a **closed circle** because {solution.sign} includes equality, so {boundary} counts."
    return f"Use an **open circle** because this strict inequality excludes {boundary}."


def graph_explanation(solution):
    comparison = "smaller" if solution.direction == "Left" else "larger"
    return f"{circle_explanation(solution)} Shade **{solution.direction.lower()}** toward {comparison} numbers."


def show_work(problem):
    st.latex(problem.latex)
    for note, equation in solving_steps(problem):
        st.write(note)
        if equation:
            st.latex(equation)


STYLE = """
<style>
main .block-container {
    max-width: 1080px;
    padding-top: 4.5rem;
    padding-bottom: 3.5rem;
}
@media (max-width: 640px) {
    main .block-container { padding: 4.25rem 1rem 2.5rem; }
}
.arcade-hero {
    position: relative;
    overflow: hidden;
    isolation: isolate;
    padding: clamp(1.35rem, 4vw, 2.65rem);
    border-radius: 1.45rem;
    color: #fbfaff;
    background: radial-gradient(ellipse at 105% 20%, #453382 0, transparent 63%),
                linear-gradient(135deg, #181535, #251d4b);
    box-shadow: 0 10px 26px #20134629;
}
.arcade-hero h1 {
    margin: .27rem 0 .55rem;
    padding: 0;
    color: #fbfaff;
    font: 800 clamp(2.02rem, 5.8vw, 3.65rem)/1.02 Inter, system-ui, sans-serif;
    letter-spacing: -.05em;
}
.arcade-hero p {
    margin: 0;
    color: #e1dbf7;
    font-size: 1rem;
    max-width: 30rem;
}
.arcade-course {
    display: inline-block;
    border-radius: 999px;
    padding: .27rem .7rem;
    background: #ddffa8;
    color: #1e2730;
    font: 750 .69rem/1.35 Inter, system-ui, sans-serif;
    letter-spacing: .1em;
    text-transform: uppercase;
}
.arcade-hero-track {
    position: absolute;
    inset: 56% 2.7rem auto auto;
    width: 25%;
    height: 2px;
    background: #a89ade7a;
    z-index: -1;
}
.arcade-hero-track::before {
    content: "";
    position: absolute;
    top: -9px;
    width: 100%;
    height: 20px;
    background: repeating-linear-gradient(to right, transparent 0 15%, #a89ade67 15% 16%, transparent 16% 20%);
}
.arcade-hero-track::after {
    content: "";
    position: absolute;
    top: -7px;
    left: 39%;
    width: 13px;
    height: 13px;
    border: 2px solid #dcfbb5;
    border-radius: 50%;
    background: #302350;
    box-shadow: 0 0 0 5px #dcfbb525;
}
@media (max-width: 800px) {
    .arcade-hero-track { opacity: .24; right: 1.2rem; }
}
@media (max-width: 500px) {
    .arcade-hero-track { display: none; }
}
.stTabs [role="tab"] { min-height: 2.7rem; }
.stTabs [role="tab"] p { font-size: 1rem; font-weight: 680; }
.st-key-studio_controls, .st-key-studio_plot, .st-key-practice_round {
    border-radius: 1rem;
}
.arcade-board {
    padding: 1rem 1rem .75rem;
    border: 1px solid #d9d4ef;
    border-radius: .94rem;
    color: #21183b;
    background: #f7f5ff;
}
.arcade-board-title {
    font: 740 .9rem/1.25 Inter, system-ui, sans-serif;
}
.arcade-track { position: relative; height: 95px; direction: ltr; }
.arcade-axis, .arcade-ray {
    position: absolute;
    top: 45px;
    height: 2px;
    left: 2%;
    right: 2%;
    background: #71658c;
}
.arcade-arrow-left, .arcade-arrow-right {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    width: 0;
    height: 0;
    border-top: 5px solid transparent;
    border-bottom: 5px solid transparent;
}
.arcade-arrow-left { left: -3px; border-right: 8px solid #71658c; }
.arcade-arrow-right { right: -3px; border-left: 8px solid #71658c; }
.arcade-tick {
    position: absolute;
    top: 39px;
    width: 2px;
    height: 14px;
    transform: translateX(-50%);
    background: #71658c;
}
.arcade-tick span {
    position: absolute;
    left: 50%;
    top: 23px;
    transform: translateX(-50%);
    color: #453a60;
    white-space: nowrap;
    font: 600 clamp(.65rem, 2vw, .85rem)/1 ui-monospace, monospace;
}
.arcade-tick.minor { top: 42px; height: 8px; }
.arcade-ray { top: 43px; height: 6px; background: #087660; }
.arcade-ray .arcade-arrow-left { border-right-color: #087660; }
.arcade-ray .arcade-arrow-right { border-left-color: #087660; }
.arcade-dot {
    position: absolute;
    top: 46px;
    width: 15px;
    height: 15px;
    box-sizing: border-box;
    transform: translate(-50%, -50%);
    border-radius: 50%;
    border: 3px solid #087660;
    background: #f7f5ff;
}
.arcade-dot.closed { background: #087660; }
.arcade-graph-summary {
    margin: 0;
    color: #413653;
    font-size: .87rem;
    overflow-wrap: anywhere;
}
.arcade-stats {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: .5rem;
}
.arcade-stat {
    border: 1px solid #dbd5ee;
    border-radius: .85rem;
    padding: .6rem clamp(.42rem, 2vw, .95rem);
    background: #f7f5ff;
    color: #292245;
}
.arcade-stat:nth-child(2) { background: #eeeafa; }
.arcade-stat:nth-child(3) { background: #edf8eb; border-color: #d0e6cb; }
.arcade-stat strong { display: block; font-size: clamp(1.14rem, 3vw, 1.55rem); line-height: 1.3; }
.arcade-stat span { display: block; font-size: .76rem; font-weight: 650; }
</style>
"""


def arrow(side):
    return f'<i class="arcade-arrow-{side.lower()}"></i>'


def number_line(title, boundary=None, closed=False, direction=None, all_values=None,
                empty_message="Choose a boundary, circle, and direction to see your graph."):
    center = 0 if boundary is None or abs(boundary) <= 3 else floor(boundary + Fraction(1, 2))
    low, high = center - 4, center + 4
    left_edge, right_edge = 2, 98

    def position(value):
        return 8 + 84 * float(Fraction(value - low) / (high - low))

    ticks = []
    for offset, value in enumerate(range(low, high + 1)):
        minor = " minor" if offset % 2 else ""
        label = "" if minor else f"<span>{number_text(value)}</span>"
        ticks.append(f'<i class="arcade-tick{minor}" style="left:{position(value):.4f}%">{label}</i>')
    overlay = ""
    if boundary is not None:
        x = position(boundary)
        left, right = (left_edge, x) if direction == "Left" else (x, right_edge)
        fill = "closed" if closed else ""
        overlay = (
            f'<div class="arcade-ray" style="left:{left:.4f}%;right:{100 - right:.4f}%">'
            f'{arrow(direction)}</div><i class="arcade-dot {fill}" style="left:{x:.4f}%"></i>'
        )
        shape = "Closed" if closed else "Open"
        summary = f"{shape} circle at {number_text(boundary)} · shaded {direction.lower()}"
    elif all_values is True:
        overlay = f'<div class="arcade-ray">{arrow("Left")}{arrow("Right")}</div>'
        summary = "All real numbers · the entire line is shaded"
    elif all_values is False:
        summary = "No solution · the line is unshaded"
    else:
        summary = empty_message

    st.html(
        f'<section class="arcade-board">'
        f'<div class="arcade-board-title">{escape(title)}</div>'
        f'<div class="arcade-track" aria-hidden="true">'
        f'<div class="arcade-axis">{arrow("Left")}{arrow("Right")}</div>{"".join(ticks)}{overlay}</div>'
        f'<p class="arcade-graph-summary" role="status">{escape(summary)}</p></section>'
    )


def solution_graph(solution):
    if isinstance(solution, Solution):
        number_line("Solution on the number line", solution.boundary, solution.closed, solution.direction)
    else:
        number_line("Solution on the number line", all_values=solution)


def try_value(problem, solution):
    with st.expander("Test a value"):
        text = st.text_input(
            "Value of x", value="0", max_chars=24, key="test_value",
            help="Try the boundary or a value on either side of it.",
        )
        try:
            x = read_number(text)
        except ValueError as error:
            st.caption(str(error))
            return
        left, right = problem.values_at(x) if problem else (x, solution.boundary)
        sign = problem.sign if problem else solution.sign
        st.latex(rf"{number_latex(left)} {TEX_SIGNS[sign]} {number_latex(right)}")
        if COMPARE[sign](left, right):
            st.success(f"x = {number_text(x)} makes the inequality true.")
        else:
            st.info(f"x = {number_text(x)} makes the inequality false.")


def graph_studio():
    st.write("Change the inequality and see how its solution moves.")
    controls, graph = st.columns([1, 1.6], gap="medium")
    problem = None
    with controls, st.container(key="studio_controls", border=True):
        mode = st.radio("Start with", ("A boundary", "A linear inequality"), horizontal=True)
        if mode == "A boundary":
            sign = st.radio("Inequality sign", SIGNS, index=3, horizontal=True, key="studio_sign")
            boundary_text = st.text_input(
                "Boundary", value="2", max_chars=24, key="studio_boundary",
                help="Enter an integer, decimal, or fraction.",
            )
            try:
                solution = Solution(read_number(boundary_text), sign)
            except ValueError as error:
                st.warning(str(error))
                solution = None
        else:
            st.latex(r"ax + b\; \square\; cx + d")
            sign = st.radio("Inequality sign", SIGNS, horizontal=True, key="linear_sign")
            left, right = st.columns(2)
            with left:
                st.caption("Left side")
                a = st.number_input("a (coefficient)", -30, 30, -2, step=1, key="left_coefficient")
                b = st.number_input("b (constant)", -30, 30, 3, step=1, key="left_constant")
            with right:
                st.caption("Right side")
                c = st.number_input("c (coefficient)", -30, 30, 0, step=1, key="right_coefficient")
                d = st.number_input("d (constant)", -30, 30, 7, step=1, key="right_constant")
            problem = Inequality(a, b, sign, c, d)
            solution = problem.solve()

    with graph, st.container(key="studio_plot", border=True):
        if solution is None:
            number_line("Number line", empty_message="Enter a valid boundary to draw the solution.")
            return
        if isinstance(solution, Solution):
            st.latex(solution.latex)
        else:
            st.markdown("**All real numbers**" if solution else "**No solution**")
        solution_graph(solution)
        if isinstance(solution, Solution):
            st.markdown(graph_explanation(solution))
        if problem:
            with st.expander("Show the algebra", expanded=True):
                show_work(problem)
        try_value(problem, solution)


@dataclass(frozen=True)
class Challenge:
    solution: Solution
    problem: Inequality | None = None


def generate_challenge(level, previous=None):
    while True:
        sign = random.choice(SIGNS)
        if level == "Graph basics":
            endpoints = [value for value in range(-8, 9)
                         if previous is None or value != previous.solution.boundary]
            return Challenge(Solution(Fraction(random.choice(endpoints)), sign))

        if level == "Solve & graph":
            coefficient, c = random.randint(1, 6), 0
        elif level == "Sign flips":
            coefficient, c = -random.randint(1, 7), 0
        else:
            coefficient = random.choice((-6, -5, -4, -3, -2, -1, 1, 2, 3, 4, 5, 6))
            c = random.choice([value for value in range(-5, 6) if value not in (0, -coefficient)])
        b = random.choice((-9, -7, -5, -4, -3, -2, 2, 3, 4, 5, 7, 9))
        if random.random() < .7:
            constant = coefficient * random.randint(-7, 7)
        else:
            constant = random.choice((-11, -9, -7, -5, -3, -1, 1, 3, 5, 7, 9, 11))
        problem = Inequality(coefficient + c, b, sign, c, b + constant)
        if previous is None or problem != previous.problem:
            return Challenge(problem.solve(), problem)


@dataclass
class Round:
    number: int
    level: str
    challenge: Challenge
    stage: str
    misses: int = 0
    hints: int = 0
    feedback: tuple = ()

    def key(self, name):
        return f"round_{self.number}_{name}"


@dataclass
class Game:
    current: Round | None = None
    score: int = 0
    streak: int = 0
    best: int = 0


def next_round(game, level):
    previous = game.current
    challenge = generate_challenge(level, previous.challenge if previous else None)
    game.current = Round(
        number=previous.number + 1 if previous else 1,
        level=level,
        challenge=challenge,
        stage="solve" if challenge.problem else "graph",
    )


def choose_level(game):
    next_round(game, st.session_state.practice_level)


def add_hint(round_):
    round_.hints = min(2, round_.hints + 1)


def miss(game, *feedback):
    game.streak = 0
    game.current.misses += 1
    game.current.feedback = ("warning", *feedback)


def check_algebra(game):
    round_ = game.current
    if round_.stage != "solve":
        return
    problem = round_.challenge.problem
    solution = round_.challenge.solution
    sign = st.session_state.get(round_.key("sign"))
    try:
        boundary = read_number(st.session_state.get(round_.key("answer"), ""))
    except ValueError as error:
        round_.feedback = ("info", str(error))
        return
    if sign is None:
        round_.feedback = ("info", "Pick the sign that belongs after x.")
        return

    feedback = []
    if boundary != solution.boundary:
        first_step, first_equation = solving_steps(problem)[0]
        feedback.append(f"Check the boundary. {first_step} Then keep isolating x.")
        if round_.misses:
            feedback.append(f"Your first line should be {inline_math(first_equation)}.")
    if sign != solution.sign:
        if (sign in ("≤", "≥")) != solution.closed:
            feedback.append("Keep the equality part of the original sign. A strict inequality stays strict, and ≤ or ≥ keeps the bar.")
        if (sign in ("<", "≤")) != (solution.direction == "Left"):
            coefficient = number_text(problem.coefficient)
            if problem.coefficient < 0:
                feedback.append(
                    f"Dividing by {coefficient} reverses {problem.sign} to {solution.sign}. "
                    "For example, 2 < 6 becomes −1 > −3 when both sides are divided by −2."
                )
            else:
                feedback.append("Adding or subtracting on both sides keeps the direction. Dividing by a positive coefficient also keeps it.")
    if feedback:
        miss(game, *feedback)
        return
    round_.stage = "graph"
    round_.hints = 0
    round_.feedback = ("success", "You isolated x. Now build its graph.")


def graph_guess(round_):
    endpoint = st.session_state.get(round_.key("endpoint"), "")
    circle = st.session_state.get(round_.key("circle"))
    direction = st.session_state.get(round_.key("direction"))
    return read_number(endpoint), circle, direction


def check_graph(game):
    round_ = game.current
    if round_.stage != "graph":
        return
    try:
        boundary, circle, direction = graph_guess(round_)
    except ValueError as error:
        round_.feedback = ("info", str(error))
        return
    if circle is None or direction is None:
        round_.feedback = ("info", "Choose a circle type and a shading direction.")
        return
    solution = round_.challenge.solution
    feedback = []
    if boundary != solution.boundary:
        feedback.append(f"Place the circle on **{number_text(solution.boundary)}**, the boundary in the solved inequality.")
    if (circle == "Closed") != solution.closed:
        feedback.append(circle_explanation(solution))
    if direction != solution.direction:
        sample = solution.boundary + (-1 if solution.direction == "Left" else 1)
        feedback.append(
            f"Try x = {number_text(sample)}. It satisfies the inequality and sits to the "
            f"**{solution.direction.lower()}** of {number_text(solution.boundary)}. Shade that way."
        )
    if feedback:
        miss(game, *feedback)
        return
    points = 50 if round_.misses else 100
    game.score += points
    if round_.misses == 0:
        game.streak += 1
        game.best = max(game.best, game.streak)
    round_.stage = "done"
    round_.feedback = ("success", f"Challenge complete! **+{points} points**")


def reveal_answer(game):
    round_ = game.current
    if round_.stage in ("solve", "graph") and round_.misses >= 2:
        round_.stage = "revealed"
        round_.feedback = ()


def inline_math(latex):
    return "$" + latex + "$"


def hints_for(round_):
    if round_.stage == "solve":
        problem = round_.challenge.problem
        first_step, _ = solving_steps(problem)[0]
        coefficient = problem.coefficient
        reduced = inline_math(inequality_latex(coefficient, 0, problem.sign, 0, problem.constant))
        first = f"Start by getting the x-term by itself. {first_step}"
        if coefficient == 1:
            return (first, f"After collecting terms, you should have {reduced}. x is already isolated.")
        rule = "Reverse the sign when you divide by a negative." if coefficient < 0 else "Keep the sign when you divide by a positive."
        return (first, f"Simplify to {reduced}. Then divide by {number_text(coefficient)}. {rule}")

    solution = round_.challenge.solution
    boundary = number_text(solution.boundary)
    return (
        "Use the number beside the inequality sign as your boundary. Does the sign allow equality?",
        f"Compare x = {number_text(solution.boundary - 1)} and x = {number_text(solution.boundary + 1)}. "
        f"Which one satisfies the inequality? Shade the side it is on relative to {boundary}.",
    )


def show_feedback(round_):
    if round_.feedback:
        kind, *messages = round_.feedback
        message = messages[0] if len(messages) == 1 else "\n".join(f"- {line}" for line in messages)
        getattr(st, kind)(message)


def practice_controls(game, round_):
    if round_.stage == "solve":
        st.markdown("**1. Isolate x**")
        st.radio("Sign after x", SIGNS, index=None, horizontal=True, key=round_.key("sign"))
        st.text_input(
            "Boundary value", placeholder="Integer, decimal, or fraction",
            max_chars=24, key=round_.key("answer"),
            help="Use a fraction such as 1/3 when a decimal repeats.",
        )
        action_label, action = "Check inequality", check_algebra
    else:
        st.markdown("**2. Build the graph**" if round_.challenge.problem else "**Build the graph**")
        st.text_input(
            "Place the circle at", placeholder="Enter a number", max_chars=24,
            key=round_.key("endpoint"), live=True,
        )
        st.radio("Circle type", ("Open", "Closed"), index=None, horizontal=True, key=round_.key("circle"))
        st.radio("Shade toward", ("Left", "Right"), index=None, horizontal=True, key=round_.key("direction"))
        action_label, action = "Check graph", check_graph

    buttons = st.container(horizontal=True, gap="small")
    buttons.button(action_label, key=round_.key("check"), on_click=action, args=(game,), type="primary")
    hint_label = "Get a hint" if round_.hints == 0 else "Get another hint"
    if round_.hints < 2:
        buttons.button(hint_label, key=round_.key("hint"), on_click=add_hint, args=(round_,))
    if round_.misses >= 2:
        buttons.button("Show answer", key=round_.key("reveal"), on_click=reveal_answer, args=(game,))
    if round_.hints:
        for index, hint in enumerate(hints_for(round_)[:round_.hints], start=1):
            st.info(f"**Hint {index}** · {hint}")
    show_feedback(round_)


def practice_game():
    if "game" not in st.session_state:
        st.session_state.game = Game()
    game = st.session_state.game

    st.html(
        '<div class="arcade-stats">'
        f'<div class="arcade-stat"><span>Score</span><strong>{game.score:,}</strong></div>'
        f'<div class="arcade-stat"><span>Current streak</span><strong>{game.streak}</strong></div>'
        f'<div class="arcade-stat"><span>Best streak</span><strong>{game.best}</strong></div>'
        '</div>'
    )
    st.caption("100 points on the first try, 50 after a retry. Hints are free. First-try wins build your streak.")
    level = st.selectbox("Challenge type", LEVELS, key="practice_level", on_change=choose_level, args=(game,))
    if game.current is None:
        next_round(game, level)
    round_ = game.current
    problem = round_.challenge.problem
    solution = round_.challenge.solution

    with st.container(key="practice_round", border=True):
        st.markdown(f"**Challenge {round_.number}** · {round_.level}")
        if round_.stage != "graph" or not problem:
            st.latex(problem.latex if problem else solution.latex)
        if round_.stage in ("done", "revealed"):
            show_feedback(round_)
            if problem:
                st.latex(solution.latex)
            solution_graph(solution)
            st.markdown(graph_explanation(solution))
            if problem:
                with st.expander("See how it works", expanded=round_.stage == "revealed"):
                    show_work(problem)
            st.button(
                "Next challenge", key=round_.key("next"), type="primary",
                on_click=next_round, args=(game, level),
            )
        elif round_.stage == "solve":
            practice_controls(game, round_)
        else:
            if problem:
                st.latex(solution.latex)
            controls, preview = st.columns([1, 1.35], gap="medium")
            with controls:
                practice_controls(game, round_)
            with preview:
                try:
                    boundary, circle, direction = graph_guess(round_)
                except ValueError:
                    boundary, circle, direction = None, None, None
                if circle and direction:
                    number_line("Your graph", boundary, circle == "Closed", direction)
                else:
                    number_line("Your graph")


def main():
    st.set_page_config(page_title="Inequality Arcade", page_icon="📐", layout="wide")
    st.html(STYLE)
    st.html(
        '<header class="arcade-hero">'
        '<div class="arcade-hero-track" aria-hidden="true"></div>'
        '<span class="arcade-course">Algebra 1</span>'
        '<h1>Inequality Arcade</h1>'
        '<p>Explore number lines and practice solving linear inequalities.</p>'
        '</header>'
    )
    studio, practice = st.tabs(("Graph Studio", "Practice Game"))
    with studio:
        graph_studio()
    with practice:
        practice_game()


if __name__ == "__main__":
    main()
