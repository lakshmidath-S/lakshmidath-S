#!/usr/bin/env python3
"""
Counts solved NeetCode problems (one folder = one problem solved) in
lakshmidath-S/neetcode-submissions, regenerates a progress-bar SVG,
and rewrites the tracker block in README.md between the markers:

    <!--NEETCODE:START--> ... <!--NEETCODE:END-->

Run by .github/workflows/neetcode-progress.yml on a schedule and on
manual dispatch, then the workflow commits any changes.
"""

import json
import os
import re
import sys
import urllib.request
from datetime import datetime, timezone

SOURCE_REPO = "lakshmidath-S/neetcode-submissions"
SOURCE_PATH = "Data Structures & Algorithms"
SOURCE_BRANCH = "main"
GOAL = 300
README_PATH = "README.md"
SVG_PATH = "assets/neetcode-progress.svg"

START_MARKER = "<!--NEETCODE:START-->"
END_MARKER = "<!--NEETCODE:END-->"


def get_solved_count() -> int:
    """Count top-level directories in SOURCE_PATH (one per solved problem).

    Does not recurse — subfolders inside a problem's folder are duplicate
    submissions, not separate problems.
    """
    from urllib.parse import quote

    url = (
        f"https://api.github.com/repos/{SOURCE_REPO}/contents/"
        f"{quote(SOURCE_PATH)}?ref={SOURCE_BRANCH}"
    )
    req = urllib.request.Request(url)
    req.add_header("Accept", "application/vnd.github+json")
    token = os.environ.get("GITHUB_TOKEN")
    if token:
        req.add_header("Authorization", f"Bearer {token}")

    with urllib.request.urlopen(req, timeout=30) as resp:
        data = json.load(resp)

    if isinstance(data, dict):
        raise RuntimeError(f"GitHub API error: {data.get('message', data)}")

    return sum(1 for item in data if item.get("type") == "dir")


def make_svg(solved: int, goal: int) -> str:
    pct = max(0.0, min(1.0, solved / goal))
    width, height = 620, 34
    bar_x, bar_y = 12, 9
    bar_w, bar_h = width - 24, 16
    fill_w = round(bar_w * pct)
    label = f"{solved} / {goal} solved  •  {round(pct * 100)}%"

    return f"""<svg xmlns="http://www.w3.org/2000/svg" width="{width}" height="{height}" viewBox="0 0 {width} {height}">
  <defs>
    <linearGradient id="fill" x1="0" y1="0" x2="1" y2="0">
      <stop offset="0%" stop-color="#00F7FF"/>
      <stop offset="100%" stop-color="#0091FF"/>
    </linearGradient>
  </defs>
  <rect x="{bar_x}" y="{bar_y}" width="{bar_w}" height="{bar_h}" rx="8" fill="#2b2b2b"/>
  <rect x="{bar_x}" y="{bar_y}" width="{fill_w}" height="{bar_h}" rx="8" fill="url(#fill)"/>
  <text x="{width / 2}" y="{height - 6}" font-family="Fira Code, Consolas, monospace" font-size="12" fill="#8b949e" text-anchor="middle">{label}</text>
</svg>"""


def update_readme(solved: int, goal: int) -> None:
    pct = round(solved / goal * 100)
    filled = round(pct / 5)
    bar_text = "█" * filled + "░" * (20 - filled)
    updated = datetime.now(timezone.utc).strftime("%Y-%m-%d")

    block = (
        f"{START_MARKER}\n"
        f'<p align="center">\n'
        f'  <img src="./assets/neetcode-progress.svg" alt="NeetCode progress: {solved}/{goal}" />\n'
        f"</p>\n"
        f'<p align="center"><code>{bar_text}</code>  <b>{solved} / {goal}</b> ({pct}%)</p>\n'
        f'<p align="center"><sub>Auto-updated from '
        f"<a href=\"https://github.com/{SOURCE_REPO}\">neetcode-submissions</a>"
        f" · last synced {updated} UTC</sub></p>\n"
        f"{END_MARKER}"
    )

    with open(README_PATH, "r", encoding="utf-8") as f:
        readme = f.read()

    pattern = re.compile(
        re.escape(START_MARKER) + r".*?" + re.escape(END_MARKER), re.DOTALL
    )
    if not pattern.search(readme):
        print("ERROR: markers not found in README.md", file=sys.stderr)
        sys.exit(1)

    readme = pattern.sub(block, readme)

    with open(README_PATH, "w", encoding="utf-8") as f:
        f.write(readme)


def main() -> None:
    solved = get_solved_count()
    print(f"Solved count: {solved}/{GOAL}")

    os.makedirs(os.path.dirname(SVG_PATH), exist_ok=True)
    with open(SVG_PATH, "w", encoding="utf-8") as f:
        f.write(make_svg(solved, GOAL))

    update_readme(solved, GOAL)


if __name__ == "__main__":
    main()
