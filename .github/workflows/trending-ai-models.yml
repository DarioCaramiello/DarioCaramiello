#!/usr/bin/env python3
"""Update a GitHub profile README with Hugging Face trending models."""
import json
import os
from datetime import datetime, timezone
from pathlib import Path
from urllib.parse import urlencode, quote
from urllib.request import Request, urlopen

START = '<!-- TRENDING-AI-MODELS:START -->'
END = '<!-- TRENDING-AI-MODELS:END -->'
README = Path(os.environ.get('PROFILE_README', 'README.md'))
COUNT = 5


def fetch_models():
    params = urlencode({'sort': 'trendingScore', 'direction': '-1', 'limit': str(COUNT)})
    request = Request('https://huggingface.co/api/models?' + params,
                      headers={'User-Agent': 'GitHub-Profile-Trending-Models/1.0', 'Accept': 'application/json'})
    with urlopen(request, timeout=25) as response:
        models = json.load(response)
    if not isinstance(models, list) or not models:
        raise ValueError('Hugging Face returned no models')
    return models[:COUNT]


def clean(value):
    return str(value or '—').replace('|', '\\|').replace('\n', ' ').replace('\r', ' ').replace('<', '&lt;').replace('>', '&gt;')


def main():
    original = README.read_text(encoding='utf-8')
    if original.count(START) != 1 or original.count(END) != 1 or original.index(START) >= original.index(END):
        raise ValueError('Expected one valid pair of README markers')

    entries = fetch_models()
    lines = [
        '| # | Model | Task | Downloads (30d) | Likes |',
        '|:--:|---|---|---:|---:|',
    ]
    for index, item in enumerate(entries, 1):
        model_id = item.get('id') or item.get('modelId')
        if not model_id or '/' not in model_id:
            raise ValueError('Unexpected model identifier in API response')
        model_name = clean(model_id)
        href = 'https://huggingface.co/' + quote(model_id, safe='/')
        task = clean((item.get('pipeline_tag') or '—').replace('-', ' ').title())
        downloads = item.get('downloads')
        likes = item.get('likes')
        display_downloads = f'{downloads:,}' if isinstance(downloads, int) else '—'
        display_likes = f'{likes:,}' if isinstance(likes, int) else '—'
        lines.append(f'| {index} | [{model_name}]({href}) | {task} | {display_downloads} | {display_likes} |')

    day = datetime.now(timezone.utc).strftime('%Y-%m-%d')
    lines += ['', f'<sub>Source: [Hugging Face Trending Models](https://huggingface.co/models?sort=trending) · Updated: {day} UTC. Downloads reflect the Hub API reporting window.</sub>']
    content = '\n'.join(lines)
    before, rest = original.split(START, 1)
    _, after = rest.split(END, 1)
    updated = before + START + '\n' + content + '\n' + END + after
    if updated != original:
        README.write_text(updated, encoding='utf-8')
        print('Updated trending models section')
    else:
        print('No changes')


if __name__ == '__main__':
    main()
