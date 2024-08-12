+++
title = 'Spark SQL'
date = 2024-07-19T23:33:57+08:00
draft = false
+++

# Spark SQL

This doc is for **spark-SQL** study ^_^

1. 由SparkSqlParser中的AstBuilder执行节点访问，将语法树的各种Context节点转换成对应的LogicalPlan节点，从而成为一棵未解析的逻辑算子树(UnresolvedLogicalPlan)，此时的逻辑算子树是最初形态，不包含数据信息与列信息等

> Context 节点是什么样的?
```text
Context 节点是ANTLR4生成的直接输出类,根据语法问价生成的.具体啥样看代码.需要熟悉ANTLR4的用法才能理解
```

> LogicalPlan节点又是什么样的?
```text
**UnresolvedLogicalPlan** 不包含 数据信息 列信息
```

> AstBuilder的注释原话
```text
The AstBuilder converts an ANTLR4 ParseTree into a catalyst Expression, LogicalPlan or TableIdentifier
```

> 入口

```scala
  override def parsePlan(sqlText: String): LogicalPlan = parse(sqlText) { parser =>
    astBuilder.visitSingleStatement(parser.singleStatement()) match {
      case plan: LogicalPlan => plan
      case _ =>
        val position = Origin(None, None)
        throw QueryParsingErrors.sqlStatementUnsupportedError(sqlText, position)
    }
  }

  重点关注 parser.singleStatement()
  ANTLR的标准用法就是
  paser.singleStatement().accept(visitor)
```

> ANTLR4 核心状态机

```java
public final SingleStatementContext singleStatement() throws RecognitionException {
		SingleStatementContext _localctx = new SingleStatementContext(_ctx, getState());
		enterRule(_localctx, 0, RULE_singleStatement);
		int _la;
		try {
			enterOuterAlt(_localctx, 1);
			{
			setState(286);
            // 核心状态转换 2000行!!!
			statement();
			setState(290);
			_errHandler.sync(this);
			_la = _input.LA(1);
			while (_la==T__0) {
				{
				{
				setState(287);
				match(T__0);
				}
				}
				setState(292);
				_errHandler.sync(this);
				_la = _input.LA(1);
			}
			setState(293);
			match(EOF);
			}
		}
		catch (RecognitionException re) {
			_localctx.exception = re;
			_errHandler.reportError(this, re);
			_errHandler.recover(this, re);
		}
		finally {
			exitRule();
		}
		return _localctx;
	}

_ctx始终表示正在解析的Context

初始状态:  parent = null   state = -1   _ctx = null

public void enterRule(ParserRuleContext localctx, int state, int ruleIndex) {
    setState(state);
    _ctx = localctx;
    _ctx.start = _input.LT(1);
    if (_buildParseTrees) addContextToParseTree();
    if ( _parseListeners != null) triggerEnterRuleEvent();
}
调用后
_ctx = localctx = new SingleStatementContext(_ctx, getState()); 
state = 0

          
```