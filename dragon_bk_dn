import json
import os
import re
from pathlib import Path

import requests
from playwright.sync_api import sync_playwright

# ===== 설정 =====
NTFY_TOPIC = os.environ["NTFY_TOPIC"]   # GitHub Secret에서 자동으로 가져옴
THEATER = "용산아이파크몰"
MOVIE = "듄-파트3"
URL = "https://cgv.co.kr/cnm/movieBook/movie"
SEEN_FILE = Path("seen_showtimes.json")
# ================


def load_seen():
    if SEEN_FILE.exists():
        return set(json.loads(SEEN_FILE.read_text(encoding="utf-8")))
    return set()


def save_seen(seen):
    SEEN_FILE.write_text(json.dumps(sorted(seen), ensure_ascii=False, indent=2), encoding="utf-8")


def send_ntfy(body):
    requests.post(f"https://ntfy.sh/{NTFY_TOPIC}", data=body.encode("utf-8"), timeout=20)


def get_showtimes():
    """TODO: 극장/영화 선택 부분은 셀렉터 정보를 받은 후 채울 예정입니다."""
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        page.goto(URL, wait_until="networkidle")
        page.wait_for_timeout(2000)
        text = page.inner_text("body")
        browser.close()
    return set(re.findall(r"\b\d{2}:\d{2}\b", text))


def main():
    seen = load_seen()
    current = get_showtimes()
    new = current - seen
    if new:
        body = (
            f"[CGV 알림] {THEATER} / {MOVIE}\n"
            f"새 상영시간: {', '.join(sorted(new))}\n"
            f"예매: {URL}"
        )
        send_ntfy(body)
        seen |= new
        save_seen(seen)
        print("알림 발송:", sorted(new))
    else:
        print("변화 없음")


if __name__ == "__main__":
    main()
