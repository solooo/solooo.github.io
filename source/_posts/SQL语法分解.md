---
title: SQL语法分解

date: 2017-03-14 22:04:39

categories:
- technology

tags:
- sql
- java
---
最近项目中需要做一个类似SQL编辑器的功能，与PL/SQL类似，用户输入不同的sql语句，后台启用jdbc去执行并返回结果。

起初想的比较简单，用户输入一条语句，提交并执行。实现起来也很简单。但是一想，如果用户输入多个SQL怎么办？于是以分号对sql进行分割，批量提交到后台执行。结果还是有问题，就是不能保证所有的语句都是select，如果是update/insert怎么办？又或者是create/drop等DML语言怎么办？
<!-- more -->
第一套方案是先以分号对sql进行分割，然后对分割后的语句，逐条使用jsqlparser进行解析，识别出select与其他非select语句。然后对select语句进行单独处理，其他非select语句，直接执行不返回结果。结果勉强能够实现简单的功能。对于select语句由于使用了远程调用的方式，一次性返回结果不能太大，需要对查询语句进行分页处理，所以自己拼装了count语句，与分页查询，可以正常返回结果。

后来又有新需求，需要在编辑器里执行存储过程的创建与执行。。。。存储过程中有分号，所以用分号分割不可行，而jsqlparser无法识别存储过程语法，无奈再次更换方案，首先要解决的是语法怎么分割的问题，对于一次性输入完成的sql来说，不存在什么问题，但在执行存储过程之前需要进行相应的设置才行，而设置需要在同一个连接中进行，所以sql需要批量同时执行，也就是一次要写多条sql，而且要正确的分割出不同的可执行的完整sql。思前想后没有什么好的解决办法，最后只能以最基本的语法，按开始与结束的字符串进行分割。

按照oracle语法分析，普通的select或insert/update/delete/create table等都是一整条完整的语句，并且以分号结束，中间不会再包含分号，所以，如果以这几个关键字开头的语句，可以直接查找分号进行分割；麻烦的问题在于处理存储过程，其中会有分号，也有其他的操作语句，但会以end字符结束，所以，分析字符串如果有创建并且是创建存储过程，则以end做为结束符时行分割，按此思路进行代码实现：

```java
/**
 * 分割sql
 */
public class SqlParserUtil {

    private static final Pattern pattern = Pattern.compile("[^A-Za-z]");

    private static final List<String> sqlKeys = new ArrayList<>();

    static  {
        sqlKeys.add("create");
        sqlKeys.add("select");
        sqlKeys.add("alter");
        sqlKeys.add("insert");
        sqlKeys.add("update");
        sqlKeys.add("drop");
        sqlKeys.add("merge");
        sqlKeys.add("truncate");
        sqlKeys.add("use");
        sqlKeys.add("set");
        sqlKeys.add("begin");
        sqlKeys.add("declare");
    }

    /**
     * 找到语句开始字符，根据语句类型判断结束字符串，遍历所有单词，直至找到结束点或语句结尾，以此分割sql
     * @param str
     * @return
     * @throws Exception
     */
    public static List<String> parserSql(String str) throws Exception {

        String sql = str.toLowerCase().replaceAll("\\s+", " ").replaceAll(";", " ; ").trim();
        String[] strs = sql.split("\\s+");

        StringBuilder sb = new StringBuilder();
        List<String> sqlList = new ArrayList<>();
        String startStr = null, endStr = null, preStr = null;
        int stack = 0;
        for (int i = 0; i < strs.length; i++) {
            boolean clearPreStr = false; // 是否将 preStr 设为null
            String s = strs[i];
            if (sb.length() == 0 && pattern.matcher(s).find()) {
                continue;
            }
            sb.append(s).append(" ");

            if (startStr == null && !sqlKeys.contains(s)) {
                int len = strs.length;
                String errorSql = "";
                while (++i < len) {
                    errorSql += strs[i];
                }
                throw new Exception("语法错误" + errorSql);
            }

            if (startStr == null && sqlKeys.contains(s)) {
                startStr = s;
                endStr = "begin".equals(s) || "declare".equals(s) ? "end" : ";"; // begin开关语句以end结尾
                stack++;
            } else if ("create".equals(startStr) && ("procedure".equals(s) || "package".equals(s)) ) {
                // 创建存储过程或包时，结束符以"end"终止
                String createStatement = sb.toString().trim();
                if (("create or replace " + s).equals(createStatement)
                        || ("create " + s).equals(createStatement)) {
                    endStr = "end";
                    if ("procedure".equals(s)) {
                        stack--;
                    }
                }

            } else if (s.equals("begin") || (!"end".equals(preStr) && "loop".equals(s))) {
                // begin/loop 关键字结束符都为 end, 所以需要+1
                stack++;
            } else if ("transaction".equals(s) && "begin".equals(preStr)) {
                stack--;
            } else if (s.equals(endStr) || i == strs.length - 1) {
                stack--;
                if (stack == 0) {
                    startStr = null;
                    clearPreStr = true;
                    if (!";".equals(endStr) && !";".equals(s)) {
                        sb.append(";");
                    }
                    sqlList.add(sb.toString());
                    sb.setLength(0);
                }
            }
            // 设置上一个关键字内容
            if (clearPreStr) {
                preStr = null;
            } else {
                preStr = s;
            }
        }

        return sqlList;

    }

    public static void main(String[] args) throws Exception {
        String sql = "select * from tt; \n select * from t2";
        List<String> sqlList = SqlParserUtil.parserSql(sql);
        for (String s : sqlList) {
            System.out.println(s);
        }
    }
}

```
以上方法，基本可以实现oracle语法各种查询，包括存储过程的分割。以后有什么更好的思路再进行更新吧，暂且如此。