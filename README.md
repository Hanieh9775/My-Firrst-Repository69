"""
Intelligent Log Analyzer
------------------------
A high-quality CLI tool for analyzing application log files.

Features:
- Automatic log level detection
- Error, warning, info statistics
- Most frequent error messages
- Clean structured output
- Single-file, production-ready design

Usage:
python log_analyzer.py app.log
"""

import sys
import re
from collections import Counter
from pathlib import Path


LOG_LEVELS = ["ERROR", "WARNING", "INFO", "DEBUG"]


class LogAnalyzer:
    def __init__(self, file_path: Path):
        self.file_path = file_path
        self.level_counter = Counter()
        self.error_messages = Counter()

    def analyze(self):
        if not self.file_path.exists():
            raise FileNotFoundError("Log file does not exist")

        with self.file_path.open("r", encoding="utf-8", errors="ignore") as file:
            for line in file:
                self._process_line(line.strip())

    def _process_line(self, line: str):
        for level in LOG_LEVELS:
            if level in line:
                self.level_counter[level] += 1
                if level == "ERROR":
                    message = self._extract_message(line)
                    self.error_messages[message] += 1
                break

    @staticmethod
    def _extract_message(line: str) -> str:
        parts = re.split(r"ERROR\s*[:-]?\s*", line, maxsplit=1)
        return parts[1].strip() if len(parts) > 1 else "Unknown error"

    def report(self):
        print("\nLog Analysis Report")
        print("=" * 30)

        total = sum(self.level_counter.values())
        print(f"Total log entries: {total}\n")

        for level in LOG_LEVELS:
            print(f"{level:<8}: {self.level_counter[level]}")

        if self.error_messages:
            print("\nMost frequent errors:")
            for msg, count in self.error_messages.most_common(5):
                print(f"- ({count}x) {msg}")


def main():
    if len(sys.argv) != 2:
        print("Usage: python log_analyzer.py <logfile>")
        sys.exit(1)

    log_file = Path(sys.argv[1])
    analyzer = LogAnalyzer(log_file)

    try:
        analyzer.analyze()
        analyzer.report()
    except Exception as exc:
        print(f"Error: {exc}")
        sys.exit(1)


if __name__ == "__main__":
    main()
