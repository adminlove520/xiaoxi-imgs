# CivilSpec 爬虫调研报告

> Date: 2026-06-09
> Status: 调研完成

---

## 1. 爬虫目标站点

### 1.1 官方权威源（优先级最高）

| 站点 | URL | 说明 | 可用性 |
|------|-----|------|--------|
| 国家标准全文公开系统 | https://openstd.samr.gov.cn | 中国官方国家标准库，32,000+ 标准可免费下载 | ✅ 推荐 |
| 全国标准信息公共服务平台 | https://spc.org.cn | 标准目录检索 | ✅ 可用 |

> ⚠️ **重要发现**：2025 年国家标准全文公开系统已开放 GB/T 推荐性标准免费下载，目前可下载 32,000+ 项国家标准，月下载量 2400 万次。

### 1.2 免费资源站（优先级中）

| 站点 | URL | 说明 | 特点 |
|------|-----|------|------|
| 图集下载网 | https://tujixiazai.com | 免注册免费下载图集 | ✅ 免费免注册 |
| 标准免费下载网 | https://www.bzmfxz.com | GB/行业标准下载 | ✅ 免费免注册 |
| 筑楼人 | https://www.zhulouren.com | 规范下载 | ✅ 有预览 |
| 久久建筑网 | https://www.99jianzhu.com | 免费建筑标准规范 | ✅ 免费 |

### 1.3 网盘资源（优先级中）

| 站点 | 类型 | 说明 | 爬取方式 |
|------|------|------|---------|
| 百度网盘 | pan.baidu.com | 大量分享资源 | 提取分享链接 + 提取码 |
| 阿里云盘 | pan.alibaba.com | 资源丰富 | API 接口 / 页面解析 |
| 文档资源站 | alipanso.com 等 | 第三方阿里云盘索引 | 页面爬取获取跳转链接 |

> 📌 **爬虫策略**：优先爬取资源站的直接下载链接；网盘只提取分享链接（不实际下载存储），作为搜索结果返回

### 1.4 目标站点分类

```
P0（必爬）：
├─ openstd.samr.gov.cn     国家标准全文公开系统（官方权威）
└─ 久久建筑网              免费规范资源

P1（定期爬）：
├─ tujixiazai.com          图集下载网
├─ bzmfxz.com             标准免费下载网
└─ zhulouren.com          筑楼人

P2（监控网盘资源）：
├─ 百度网盘分享链接        提取链接存入 metadata
└─ 阿里云盘资源索引站      提取跳转链接
```

---

## 2. 爬虫技术方案

### 2.1 技术栈

```python
# 核心依赖
requests          # HTTP 请求
beautifulsoup4    # HTML 解析
playwright        # 动态页面渲染（JavaScript 加载）
scrapy            # 分布式爬虫框架（可选）
lxml              # 高性能 XML/HTML 解析
tenacity          # 重试机制
```

### 2.2 爬虫架构

```python
civil-spec-crawler/
├── crawlers/
│   ├── __init__.py
│   ├── base.py           # 爬虫基类（统一重试、限速、错误处理）
│   ├── openstd.py        # 国家标准全文公开系统
│   ├── tujixiazai.py     # 图集下载网
│   ├── bzmfxz.py         # 标准免费下载网
│   ├── zhulouren.py      # 筑楼人
│   └── netdisk.py        # 网盘资源提取器
├── parsers/
│   ├── __init__.py
│   ├── metadata.py       # 元数据解析
│   └── pdf_analyzer.py   # PDF 初步分析（大小、页数）
├── storage/
│   ├── __init__.py
│   └── github_sync.py    # GitHub 仓库同步
├── config.py             # 配置文件
├── main.py               # 入口
└── requirements.txt
```

### 2.3 核心爬虫实现

#### 2.3.1 基类（统一限速 + 重试）

```python
# crawlers/base.py
import time
import requests
from tenacity import retry, stop_after_attempt, wait_exponential
from typing import List, Dict, Optional

class BaseCrawler:
    def __init__(self, name: str, delay: float = 1.0):
        self.name = name
        self.delay = delay  # 请求间隔（秒）
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        })

    @retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
    def fetch(self, url: str, **kwargs) -> Optional[requests.Response]:
        time.sleep(self.delay)
        resp = self.session.get(url, timeout=30, **kwargs)
        resp.raise_for_status()
        return resp

    def parse_metadata(self, html: str) -> List[Dict]:
        raise NotImplementedError
```

#### 2.3.2 国家标准全文公开系统爬虫

```python
# crawlers/openstd.py
from crawlers.base import BaseCrawler
from bs4 import BeautifulSoup
from typing import List, Dict
import re

class OpenStdCrawler(BaseCrawler):
    """国家标准全文公开系统爬虫 - openstd.samr.gov.cn"""

    BASE_URL = "https://openstd.samr.gov.cn"
    SEARCH_URL = f"{BASE_URL}/bzgk/gb/index"

    def search(self, keyword: str, page: int = 1) -> List[Dict]:
        """搜索标准"""
        url = f"{self.SEARCH_URL}?keyword={keyword}&page={page}"
        resp = self.fetch(url)
        return self.parse_search_results(resp.text)

    def parse_search_results(self, html: str) -> List[Dict]:
        """解析搜索结果"""
        soup = BeautifulSoup(html, 'lxml')
        items = []

        # 国家标准全文公开系统页面结构
        for row in soup.select('.gb-list tr'):
            link = row.select_one('a[href*="/gb/"]')
            if not link:
                continue

            spec_id = re.search(r'GB[/\w]+', link.text)
            items.append({
                'id': spec_id.group() if spec_id else '',
                'name': link.text.strip(),
                'url': self.BASE_URL + link['href'],
                'category': '规范',
                'status': row.select_one('.status') or '现行',
            })

        return items

    def get_detail(self, url: str) -> Dict:
        """获取标准详情页"""
        resp = self.fetch(url)
        soup = BeautifulSoup(resp.text, 'lxml')

        # 提取标准详情
        return {
            'title': soup.select_one('h1').text.strip(),
            'issue_date': soup.select_one('[itemprop="issueDate"]') or '',
            'pages': soup.select_one('.pages') or '',
            'pdf_url': self._extract_pdf_url(soup),
        }

    def _extract_pdf_url(self, soup: BeautifulSoup) -> str:
        """提取 PDF 下载链接"""
        download_btn = soup.select_one('a[href*=".pdf"]')
        return download_btn['href'] if download_btn else ''
```

#### 2.3.3 图集下载网爬虫

```python
# crawlers/tujixiazai.py
from crawlers.base import BaseCrawler
from bs4 import BeautifulSoup
from typing import List, Dict
import re

class TujixiazaiCrawler(BaseCrawler):
    """图集下载网爬虫 - tujixiazai.com"""

    BASE_URL = "https://tujixiazai.com"
    CATEGORIES = {
        '规范': '/biaozhunguifan/',
        '图集': '/tujijixiazai/',
        '标准': '/hangyebiaozhun/',
    }

    def crawl_category(self, category: str = '规范', pages: int = 5) -> List[Dict]:
        """爬取分类下的所有页面"""
        all_items = []
        for page in range(1, pages + 1):
            url = f"{self.BASE_URL}{self.CATEGORIES.get(category, '')}list_{page}.html"
            try:
                resp = self.fetch(url)
                items = self.parse_list(resp.text)
                all_items.extend(items)
            except Exception as e:
                print(f"Page {page} failed: {e}")
        return all_items

    def parse_list(self, html: str) -> List[Dict]:
        """解析列表页"""
        soup = BeautifulSoup(html, 'lxml')
        items = []

        for item in soup.select('.list-item'):
            title_elem = item.select_one('.title a')
            if not title_elem:
                continue

            items.append({
                'id': self._extract_spec_id(title_elem.text),
                'name': title_elem.text.strip(),
                'url': title_elem['href'],
                'size': item.select_one('.size') or '',
                'category': '图集',
            })

        return items

    def _extract_spec_id(self, text: str) -> str:
        """从标题提取规范编号"""
        patterns = [
            r'\d{2}G\d{3}',       # 如：16G101
            r'GB\d+[/-]\d+',       # 如：GB50010-2010
            r'JGJ\d+[/-]\d+',      # 如：JGJ120-2012
        ]
        for pattern in patterns:
            match = re.search(pattern, text)
            if match:
                return match.group()
        return ''
```

#### 2.3.4 网盘链接提取器

```python
# crawlers/netdisk.py
from crawlers.base import BaseCrawler
from bs4 import BeautifulSoup
import re

class NetdiskCrawler(BaseCrawler):
    """网盘资源提取器 - 百度网盘、阿里云盘"""

    def search_baidu(self, keyword: str) -> List[Dict]:
        """搜索百度网盘资源"""
        # 方式1：通过第三方索引站搜索
        search_url = f"https://www.wangpansou.cn/s.php?q={keyword}&wp=0"

        try:
            resp = self.fetch(search_url)
            return self._parse_pan_results(resp.text)
        except:
            return []

    def search_alipan(self, keyword: str) -> List[Dict]:
        """搜索阿里云盘资源"""
        # 通过 alipanso.com 搜索
        search_url = f"https://www.alipanso.com/search.html?kw={keyword}"

        try:
            resp = self.fetch(search_url)
            return self._parse_alipan_results(resp.text)
        except:
            return []

    def _parse_pan_results(self, html: str) -> List[Dict]:
        """解析百度网盘搜索结果"""
        soup = BeautifulSoup(html, 'lxml')
        results = []

        for item in soup.select('.result-item'):
            title = item.select_one('.title')
            link_elem = item.select_one('a[href*="pan.baidu.com"]')

            if title and link_elem:
                # 提取分享链接和提取码
                share_url = link_elem['href']
                pwd = item.select_one('.pwd') or ''

                results.append({
                    'name': title.text.strip(),
                    'share_url': share_url,
                    'pwd': pwd,
                    'source': '百度网盘',
                })

        return results

    def _parse_alipan_results(self, html: str) -> List[Dict]:
        """解析阿里云盘搜索结果"""
        soup = BeautifulSoup(html, 'lxml')
        results = []

        for item in soup.select('.resource-item'):
            link = item.select_one('a')
            if not link:
                continue

            results.append({
                'name': link.text.strip(),
                'share_url': link['href'],
                'source': '阿里云盘',
            })

        return results
```

### 2.4 GitHub Actions Workflow

```yaml
# .github/workflows/crawl.yml
name: 规范爬虫

on:
  schedule:
    # 每天凌晨3点运行（避开高峰期）
    - cron: '0 3 * * *'
  workflow_dispatch:  # 支持手动触发
    inputs:
      category:
        description: '爬取分类'
        required: true
        default: 'all'
        type: choice
        options:
          - all
          - 规范
          - 图集
          - 标准

jobs:
  crawl-and-sync:
    runs-on: ubuntu-latest
    timeout-minutes: 120

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Run crawler
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          python main.py --category ${{ inputs.category || 'all' }}

      - name: Commit and push
        run: |
          git config user.name "CivilSpec Bot"
          git config user.email "bot@civilspec.com"

          if git diff --exit-code; then
            echo "No changes to commit"
          else
            git add -A
            git commit -m "chore: 更新规范库 $(date +%Y-%m-%d)"
            git push
          fi
```

### 2.5 爬虫主入口

```python
# main.py
import argparse
import json
from pathlib import Path
from crawlers.openstd import OpenStdCrawler
from crawlers.tujixiazai import TujixiazaiCrawler
from crawlers.bzmfxz import BzmfxzCrawler
from crawlers.zhulouren import ZhulourenCrawler
from crawlers.netdisk import NetdiskCrawler

def main():
    parser = argparse.ArgumentParser(description='CivilSpec 爬虫')
    parser.add_argument('--category', default='all', choices=['all', '规范', '图集', '标准'])
    parser.add_argument('--keywords', nargs='+', default=['GB', '16G101', 'JGJ'])
    parser.add_argument('--output', default='data/specs.json')
    args = parser.parse_args()

    all_specs = []

    # 爬取国家标准全文公开系统
    print("正在爬取国家标准全文公开系统...")
    openstd = OpenStdCrawler(delay=2.0)
    for kw in args.keywords:
        try:
            specs = openstd.search(kw)
            all_specs.extend(specs)
            print(f"  关键词 {kw}: 获取 {len(specs)} 条")
        except Exception as e:
            print(f"  关键词 {kw} 失败: {e}")

    # 爬取图集下载网
    if args.category in ['all', '图集']:
        print("正在爬取图集下载网...")
        crawler = TujixiazaiCrawler(delay=1.5)
        specs = crawler.crawl_category('图集', pages=10)
        all_specs.extend(specs)
        print(f"  图集: 获取 {len(specs)} 条")

    # 爬取网盘资源
    print("正在提取网盘资源...")
    netdisk = NetdiskCrawler(delay=2.0)
    for kw in args.keywords:
        try:
            baidu = netdisk.search_baidu(kw)
            alipan = netdisk.search_alipan(kw)
            print(f"  {kw}: 百度网盘 {len(baidu)} 条, 阿里云盘 {len(alipan)} 条")
        except Exception as e:
            print(f"  {kw} 网盘搜索失败: {e}")

    # 去重
    seen = set()
    unique_specs = []
    for spec in all_specs:
        if spec['id'] not in seen:
            seen.add(spec['id'])
            unique_specs.append(spec)

    # 保存
    output = Path(args.output)
    output.parent.mkdir(parents=True, exist_ok=True)
    output.write_text(json.dumps(unique_specs, ensure_ascii=False, indent=2))
    print(f"\n完成！共获取 {len(unique_specs)} 条规范")

if __name__ == '__main__':
    main()
```

---

## 3. 版权与合规

### 3.1 版权说明

| 类型 | 处理策略 |
|------|---------|
| 国家标准（GB）全文公开部分 | ✅ 可直接下载存储到 GitHub |
| 行业标准（JGJ） | ⚠️ 看具体平台政策，优先只存元数据 |
| 图集（16G101 等） | ⚠️ 版权复杂，优先只存下载链接 |
| 网盘资源 | ⚠️ 只提取链接，不存储实际文件 |

### 3.2 合规建议

1. **GitHub 仓库设为 Private** — 避免版权纠纷
2. **元数据 + 链接优先** — 不存储实际 PDF 文件
3. **robots.txt 遵守** — 爬虫遵守目标站点的 robots.txt
4. **限速** — 请求间隔 ≥ 1 秒
5. **User-Agent 伪装** — 使用正常浏览器 UA
6. **来源标注** — 每个规范条目标注来源站点

---

## 4. 爬虫目标清单

| 规范类别 | 代表规范 | 目标站点 | 存储方式 |
|---------|---------|---------|---------|
| 混凝土结构 | GB 50010 | openstd.samr.gov.cn | 实际 PDF |
| 钢筋图集 | 16G101 | tujixiazai.com | 下载链接 |
| 建筑结构荷载 | GB 50009 | openstd.samr.gov.cn | 实际 PDF |
| 砌体结构 | GB 50203 | bzmfxz.com | 下载链接 |
| 地基基础 | GB 50007 | openstd.samr.gov.cn | 实际 PDF |
| 抗震规范 | GB 50011 | openstd.samr.gov.cn | 实际 PDF |
| 施工规范 | JGJ 120 | openstd.samr.gov.cn | 实际 PDF |
| 网盘分享资源 | 各类 | 百度/阿里云盘 | 链接 |

---

*调研完成，待 Claude Code 开发实施*
