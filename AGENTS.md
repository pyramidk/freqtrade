# Repository Guidelines

## 项目结构与模块组织
核心交易代码位于 `freqtrade/`，涵盖策略执行、任务调度、REST API 与 CLI 入口，新增功能请先梳理对应服务层。`user_data/` 是挂载给本地与容器的工作区，保存策略、配置、日志，请勿提交密钥。`config_examples/` 提供最新配置模板，扩展字段时同步示例与文档。`tests/` 使用 pytest 依模块拆分，便于针对性回归；`docs/` 与 `mkdocs.yml` 驱动官方站点内容。Python SDK 与客户端示例位于 `ft_client/`，其中 `freqtrade_client/` 对应 API 封装，`test_client/` 演示常见调用。运维脚本集中在 `scripts/`、`build_helpers/`，容器镜像和扩展位于 `docker/` 与 `docker-compose.yml`。

## 构建、测试与开发命令
```bash
python -m venv .venv && source .venv/bin/activate     # 建议的虚拟环境
pip install -r requirements.txt                       # 安装运行依赖
pip install -r requirements-dev.txt                   # 安装开发与测试工具
pre-commit install && pre-commit run -a               # 本地执行格式与静态检查
pytest                                                # 运行全部测试
pytest tests/test_edge/test_edge_cli.py               # 针对性复现示例
ruff check . && ruff format .                         # 统一格式与静态规则
mypy freqtrade                                        # 校验类型提示
docker compose up freqtrade                           # 以容器模式验证交易循环
```

## 编码风格与命名约定
Python 代码统一四空格缩进、类型注解齐全，公共接口需带 reST 风格中文 docstring（双引号开头）。变量、模块、策略姓名均使用英文 snake_case，类名使用 PascalCase。格式化由 `ruff format` 驱动，避免手工对齐；静态检查默认开启 `ruff`、`mypy`、`codespell`。策略配置与模板命名遵循 `SampleStrategy`、`MyStrategyV2` 等驼峰模式，JSON/YAML 键使用小写加连字符。引入新第三方库时同步更新 `pyproject.toml` 与相关 `requirements-*.txt`，并在 PR 中说明兼容性考量。

## 测试指引
新增功能需提供正反向用例，覆盖主要分支并保持 Coveralls 覆盖率不下降。默认运行 `pytest`，需要加速时可使用 `pytest -n auto --dist loadfile`。单文件或单用例可用 `pytest tests/test_xxx.py::test_case` 精确定位。涉及异步或 REST 逻辑时，参考 `tests/rpc/` 与 `tests/webserver/` 的 fixture。需要输出报告时使用 `pytest --cov=freqtrade --cov-report=term-missing` 并附在 PR 描述。复杂策略建议结合 `user_data/strategies` 中的示例行情数据编写回测用例，便于复现真实交易路径。

## 提交与 Pull Request 指南
所有更改基于 `develop` 分支，提交信息使用英文祈使句（例如 `Fix dry-run wallet sync`），首行不超过 72 个字符并在正文引用 `Refs #1234`。提交前确保 `pre-commit run -a`、`pytest` 与核心类型检查通过。PR 需包含：变更摘要、动机、测试结果、配置或文档更新截图（若涉及 UI/API）。涉及 breaking change 时在描述中注明迁移步骤，并同步更新 `docs/` 与配置示例。

## 安全与配置提示
私钥、API Key、数据库 URL 等敏感信息仅保留在 `.env` 或 `user_data/`，使用 `config_examples/config_full.json` 作为对照而非直接提交。部署容器前确认 `user_data/logs/` 与数据库路径已挂载，避免覆盖生产数据。运行时若需暴露 REST API，请阅读 `docs/rest-api.md` 并启用身份认证，避免将接口直接暴露公网。依赖新增前评估许可证与体积，对频繁更新的依赖在 PR 中说明锁定策略。
