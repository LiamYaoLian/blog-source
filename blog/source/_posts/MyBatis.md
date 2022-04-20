---
title: MyBatis
date: 2022-04-19 17:12:44
tags:
---

## Steps
### Dependency
### mybatis-config.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- goods_id ==> goodsId 驼峰命名转换 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- 配置SLF4J日志组件 -->
        <setting name="logImpl" value="SLF4J"/>
    </settings>
    <!--启用Pagehelper分页插件-->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <!--设置数据库类型-->
            <property name="helperDialect" value="mysql"/>
            <!--分页合理化-->
            <property name="reasonable" value="true"/>
        </plugin>
    </plugins>

    <!--设置默认指向的数据库-->
    <environments default="dev">
        <!--配置环境，不同的环境不同的id名字-->
        <environment id="dev">
            <!-- 采用JDBC方式对数据库事务进行commit/rollback -->
            <transactionManager type="JDBC"></transactionManager>
            <!--采用连接池方式管理数据库连接-->
            <!--<dataSource type="POOLED">-->
            <dataSource type="com.ll.mybatis.datasource.C3P0DataSourceFactory">
                <property name="driverClass" value="com.mysql.jdbc.Driver"/>
                <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/babytun?useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="user" value="root"/>
                <property name="password" value="root"/>
                <property name="initialPoolSize" value="5"/>
                <property name="maxPoolSize" value="20"/>
                <property name="minPoolSize" value="5"/>
                <!--...-->
            </dataSource>
        </environment>
        <environment id="prd">
            <!-- 采用JDBC方式对数据库事务进行commit/rollback -->
            <transactionManager type="JDBC"></transactionManager>
            <!--采用连接池方式管理数据库连接-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://192.168.1.155:3306/babytun?useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mappers/goods.xml"/>
        <mapper resource="mappers/goods_detail.xml"/>
    </mappers>
</configuration>
```
### Entity

### Mapper
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="goods">
    <!--开启了二级缓存
        eviction是缓存的清除策略,当缓存对象数量达到上限后,自动触发对应算法对缓存对象清除
            1.LRU – 最近最久未使用:移除最长时间不被使用的对象。
            O1 O2 O3 O4 .. O512
            14 99 83 1     893
            2.FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
            3.SOFT – 软引用:移除基于垃圾收集器状态和软引用规则的对象。
            4.WEAK – 弱引用:更积极的移除基于垃圾收集器状态和弱引用规则的对象。
        flushInterval 间隔多久自动清空缓存，单位毫秒
        size 保存对象或集合（1个集合算一个对象）的上限
        readOnly true代表从缓存取出对象本身，false代表从缓存取出对象的副本
    -->
    <cache eviction="LRU" flushInterval="600000" size="512" readOnly="true"/>
    <select id="selectAll" resultType="com.ll.mybatis.entity.Goods" useCache="false">
        select * from t_goods order by goods_id desc limit 10
    </select>
    <!-- 单参数传递,使用parameterType指定参数的数据类型即可,SQL中#{value}提取参数-->
    <select id="selectById" parameterType="Integer" resultType="com.ll.mybatis.entity.Goods">
        select * from t_goods where  goods_id = #{value}
    </select>

    <!-- 多参数传递时,使用parameterType指定Map接口,SQL中#{key}提取参数 -->
    <select id="selectByPriceRange" parameterType="java.util.Map" resultType="com.ll.mybatis.entity.Goods">
        select * from t_goods
        where
          current_price between  #{min} and #{max}
        order by current_price
        limit 0,#{limt}
    </select>

    <!-- 利用LinkedHashMap保存多表关联结果
        MyBatis会将每一条记录包装为LinkedHashMap对象
        key是字段名  value是字段对应的值 , 字段类型根据表结构进行自动判断
        优点: 易于扩展,易于使用
        缺点: 太过灵活,无法进行编译时检查
     -->
    <select id="selectGoodsMap" resultType="java.util.LinkedHashMap" flushCache="true">
        select g.* , c.category_name,'1' as test from t_goods g , t_category c
        where g.category_id = c.category_id
    </select>

    <!--结果映射-->
    <resultMap id="rmGoods" type="com.ll.mybatis.dto.GoodsDTO">
        <!--设置主键字段与属性映射-->
        <id property="goods.goodsId" column="goods_id"></id>
        <!--设置非主键字段与属性映射-->
        <result property="goods.title" column="title"></result>
        <result property="goods.originalCost" column="original_cost"></result>
        <result property="goods.currentPrice" column="current_price"></result>
        <result property="goods.discount" column="discount"></result>
        <result property="goods.isFreeDelivery" column="is_free_delivery"></result>
        <result property="goods.categoryId" column="category_id"></result>
        <result property="category.categoryId" column="category_id"></result>
        <result property="category.categoryName" column="category_name"></result>
        <result property="category.parentId" column="parent_id"></result>
        <result property="category.categoryLevel" column="category_level"></result>
        <result property="category.categoryOrder" column="category_order"></result>


        <result property="test" column="test"/>
    </resultMap>
    <select id="selectGoodsDTO" resultMap="rmGoods">
        select g.* , c.*,'1' as test from t_goods g , t_category c
        where g.category_id = c.category_id
    </select>
    <!--flushCache="true"在sql执行后强制清空缓存-->
    <insert id="insert" parameterType="com.ll.mybatis.entity.Goods" flushCache="true">
        INSERT INTO t_goods(title, sub_title, original_cost, current_price, discount, is_free_delivery, category_id)
        VALUES (#{title} , #{subTitle} , #{originalCost}, #{currentPrice}, #{discount}, #{isFreeDelivery}, #{categoryId})
      <!--<selectKey resultType="Integer" keyProperty="goodsId" order="AFTER">-->
          <!--select last_insert_id()-->
      <!--</selectKey>-->
      <!--selectKey applies to all relational databases; useGeneratedKeys applies to databases that support auto increment PK-->
      <!--Oracle: doesn't have auto increment PK-->
      <!--<selectKey resultType="Integer" keyProperty="goodsId" order="BEFORE">-->
          <!--select seq_goods.nextval as id from dual-->
      <!--</selectKey>-->
    </insert>

    <update id="update" parameterType="com.ll.mybatis.entity.Goods">
        UPDATE t_goods
        SET
          title = #{title} ,
          sub_title = #{subTitle} ,
          original_cost = #{originalCost} ,
          current_price = #{currentPrice} ,
          discount = #{discount} ,
          is_free_delivery = #{isFreeDelivery} ,
          category_id = #{categoryId}
        WHERE
          goods_id = #{goodsId}
    </update>
    <!--delete from t_goods where goods_id in (1920,1921)-->
    <delete id="delete" parameterType="Integer">
        delete from t_goods where goods_id = #{value}
    </delete>

    <select id="selectByTitle" parameterType="java.util.Map" resultType="com.ll.mybatis.entity.Goods">
        select * from t_goods where title = #{title}
        ${order}
    </select>

    <select id="dynamicSQL" parameterType="java.util.Map" resultType="com.ll.mybatis.entity.Goods">
        select * from t_goods
        <where>
          <if test="categoryId != null">
              and category_id = #{categoryId}
          </if>
          <if test="currentPrice != null">
              and current_price &lt; #{currentPrice}
          </if>
        </where>
    </select>

    <!--
        resultMap可用于说明一对多或者多对一的映射逻辑
        id 是resultMap属性引用的标志
        type 指向One的实体(Goods)
    -->
    <resultMap id="rmGoods1" type="com.ll.mybatis.entity.Goods">
        <!-- 映射goods对象的主键到goods_id字段 -->
        <id column="goods_id" property="goodsId"></id>
        <!--
            collection的含义是,在
            select * from t_goods limit 0,1 得到结果后,对所有Goods对象遍历得到goods_id字段值,
            并代入到goodsDetail命名空间的findByGoodsId的SQL中执行查询,
            将得到的"商品详情"集合赋值给goodsDetails List对象.
        -->
        <collection property="goodsDetails" select="goodsDetail.selectByGoodsId"
                    column="goods_id"/>
    </resultMap>
    <select id="selectOneToMany" resultMap="rmGoods1">
        select * from t_goods limit 0,10
    </select>

    <select id="selectPage" resultType="com.ll.mybatis.entity.Goods">
        select * from t_goods where current_price &lt; 1000
    </select>

    <!--INSERT INTO table-->
    <!--VALUES ("a" , "a1" , "a2"),("b" , "b1" , "b2"),(....)-->
    <!--Pro: faster than inserting one. -->
    <!--limitation: 1. cannot get the id of the inserted row. 2. If the SQL query is too long, the server may reject it. Need stress test. -->
    <insert id="batchInsert" parameterType="java.util.List">
        INSERT INTO t_goods(title, sub_title, original_cost, current_price, discount, is_free_delivery, category_id)
        VALUES
        <foreach collection="list" item="item" index="index" separator=",">
            (#{item.title},#{item.subTitle}, #{item.originalCost}, #{item.currentPrice}, #{item.discount}, #{item.isFreeDelivery}, #{item.categoryId})
        </foreach>
    </insert>
    <!--in (1901,1902)-->
    <delete id="batchDelete" parameterType="java.util.List">
        DELETE FROM t_goods WHERE goods_id in
        <foreach collection="list" item="item" index="index" open="(" close=")" separator=",">
            #{item}
        </foreach>
    </delete>
</mapper>
```
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="goodsDetail">
    <select id="selectByGoodsId" parameterType="Integer"
            resultType="com.ll.mybatis.entity.GoodsDetail">
        select * from t_goods_detail where goods_id = #{value}
    </select>

    <resultMap id="rmGoodsDetail" type="com.ll.mybatis.entity.GoodsDetail">
        <id column="gd_id" property="gdId"/>
        <result column="goods_id" property="goodsId"/>
        <association property="goods" select="goods.selectById" column="goods_id"></association>
    </resultMap>
    <select id="selectManyToOne" resultMap="rmGoodsDetail">
        select * from t_goods_detail limit 0,20
    </select>
</mapper>
```

### SqlSessionFactory
* a Mybatis class used to initialize MyBatis and create SqlSession objects
* Singleton
* SqlSession uses JDBC
```
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.Reader;

/**
 * MyBatisUtils工具类,创建全局唯一的SqlSessionFactory对象
 */
public class MyBatisUtils {
    //利用static(静态)属于类不属于对象,且全局唯一
    private static SqlSessionFactory sqlSessionFactory = null;
    //利用静态块在初始化类时实例化sqlSessionFactory
    static {
        Reader reader = null;
        try {
            reader = Resources.getResourceAsReader("mybatis-config.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
        } catch (IOException e) {
            e.printStackTrace();
            //初始化错误时,通过抛出异常ExceptionInInitializerError通知调用者
            throw new ExceptionInInitializerError(e);
        }
    }

    /**
     * openSession 创建一个新的SqlSession对象
     * @return SqlSession对象
     */
    public static SqlSession openSession(){
        return sqlSessionFactory.openSession();
    }

    /**
     * 释放一个有效的SqlSession对象
     * @param session 准备释放SqlSession对象
     */
    public static void closeSession(SqlSession session){
        if(session != null){
            session.close();
        }
    }
}
```
### Manipulate using SqlSession
```
package com.ll.mybatis;

import com.github.pagehelper.Page;
import com.github.pagehelper.PageHelper;
import com.ll.mybatis.dto.GoodsDTO;
import com.ll.mybatis.entity.Goods;
import com.ll.mybatis.entity.GoodsDetail;
import com.ll.mybatis.utils.MyBatisUtils;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.Reader;
import java.sql.Connection;
import java.util.*;

//JUNIT单元测试类
public class MyBatisTestor {
    /**
     * 初始化SqlSessionFactory
     * @throws IOException
     */
    @Test
    public void testSqlSessionFactory() throws IOException {
        //利用Reader加载classpath下的mybatis-config.xml核心配置文件
        Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
        //初始化SqlSessionFactory对象,同时解析mybatis-config.xml文件
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
        System.out.println("SessionFactory加载成功");
        SqlSession sqlSession = null;
        try {
            //创建SqlSession对象,SqlSession是JDBC的扩展类,用于与数据库交互
            sqlSession = sqlSessionFactory.openSession();
            //创建数据库连接(测试用)
            Connection connection = sqlSession.getConnection();
            System.out.println(connection);
        } catch (Exception e){
            e.printStackTrace();
        } finally {
            if (sqlSession != null){
                //如果type="POOLED",代表使用连接池,close则是将连接回收到连接池中
                //如果type="UNPOOLED",代表直连,close则会调用Connection.close()方法关闭连接
                sqlSession.close();
            }
        }
    }

    /**
     * MyBatisUtils使用指南
     * @throws Exception
     */
    @Test
    public void testMyBatisUtils() throws Exception {
        SqlSession sqlSession = null;
        try {
            sqlSession = MyBatisUtils.openSession();
            Connection connection = sqlSession.getConnection();
            System.out.println(connection);
        } catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(sqlSession);
        }
    }

    /**
     * select查询语句执行
     * @throws Exception
     */
    @Test
    public void testSelectAll() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            List<Goods> list = session.selectList("goods.selectAll");
            for(Goods g : list){
                System.out.println(g.getTitle());
            }
        } catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 传递单个SQL参数
     * @throws Exception
     */
    @Test
    public void testSelectById() throws Exception {
        SqlSession session = null;
        try{
            session = MyBatisUtils.openSession();
            Goods goods = session.selectOne("goods.selectById" , 1603);
            System.out.println(goods.getTitle());
        }catch (Exception e){
            throw e;
        }finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 传递多个SQL参数
     * @throws Exception
     */
    @Test
    public void testSelectByPriceRange() throws Exception {
        SqlSession session = null;
        try{
            session = MyBatisUtils.openSession();
            Map param = new HashMap();
            param.put("min",100);
            param.put("max" , 500);
            param.put("limt" , 10);
            List<Goods> list = session.selectList("goods.selectByPriceRange", param);
            for(Goods g:list){
                System.out.println(g.getTitle() + ":" + g.getCurrentPrice());

            }
        }catch (Exception e){
            throw e;
        }finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 利用Map接收关联查询结果
     * @throws Exception
     */
    @Test
    public void testSelectGoodsMap() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            List<Map> list = session.selectList("goods.selectGoodsMap");
            for(Map map : list){
                System.out.println(map);
            }
        } catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 利用ResultMap进行结果映射
     * @throws Exception
     */
    @Test
    public void testSelectGoodsDTO() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            List<GoodsDTO> list = session.selectList("goods.selectGoodsDTO");
            for (GoodsDTO g : list) {
                System.out.println(g.getGoods().getTitle());
            }
        } catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 新增数据
     * @throws Exception
     */
    @Test
    public void testInsert() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            Goods goods = new Goods();
            goods.setTitle("测试商品");
            goods.setSubTitle("测试子标题");
            goods.setOriginalCost(200f);
            goods.setCurrentPrice(100f);
            goods.setDiscount(0.5f);
            goods.setIsFreeDelivery(1);
            goods.setCategoryId(43);
            //insert()方法返回值代表本次成功插入的记录总数
            int num = session.insert("goods.insert", goods);
            session.commit();//提交事务数据
            System.out.println(goods.getGoodsId());
        } catch (Exception e){
            if(session != null){
                session.rollback();//回滚事务
            }
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 更新数据
     * @throws Exception
     */
    @Test
    public void testUpdate() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            Goods goods = session.selectOne("goods.selectById", 739);
            goods.setTitle("更新测试商品");
            int num = session.update("goods.update" , goods);
            session.commit();//提交事务数据
        } catch (Exception e){
            if(session != null){
                session.rollback();//回滚事务
            }
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 删除数据
     * @throws Exception
     */
    @Test
    public void testDelete() throws Exception {
        SqlSession session = null;
        try{
            session = MyBatisUtils.openSession();
            int num = session.delete("goods.delete" , 739);
            session.commit();//提交事务数据
        } catch (Exception e){
            if(session != null){
                session.rollback();//回滚事务
            }
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 预防SQL注入
     * @throws Exception
     */
    @Test
    public void testSelectByTitle() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            Map param = new HashMap();
            /*
                ${}原文传值
                select * from t_goods
                where title = '' or 1 =1 or title = '【德国】爱他美婴幼儿配方奶粉1段800g*2罐 铂金版'
            */
            /*
               #{}预编译
               select * from t_goods
                where title = "'' or 1 =1 or title = '【德国】爱他美婴幼儿配方奶粉1段800g*2罐 铂金版'"
            */

            param.put("title","'' or 1=1 or title='【德国】爱他美婴幼儿配方奶粉1段800g*2罐 铂金版'");
            param.put("order" , " order by title desc");
            List<Goods> list = session.selectList("goods.selectByTitle", param);
            for(Goods g:list){
                System.out.println(g.getTitle() + ":" + g.getCurrentPrice());
            }
        } catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 动态SQL语句
     * @throws Exception
     */
    @Test
    public void testDynamicSQL() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            Map param = new HashMap();
            param.put("categoryId", 44);
            param.put("currentPrice", 500);
            //查询条件
            List<Goods> list = session.selectList("goods.dynamicSQL", param);
            for(Goods g:list){
                System.out.println(g.getTitle() + ":" +
                        g.getCategoryId()  + ":" + g.getCurrentPrice());

            }
        } catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 测试一级缓存
     * @throws Exception
     */
    @Test
    public void testLv1Cache() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            Goods goods = session.selectOne("goods.selectById" , 1603);
            Goods goods1 = session.selectOne("goods.selectById" , 1603);
            System.out.println(goods.hashCode() + ":" + goods1.hashCode());
        } catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }

        try {
            session = MyBatisUtils.openSession();
            Goods goods = session.selectOne("goods.selectById" , 1603);
            session.commit();//commit提交时对该namespace缓存强制清空
            Goods goods1 = session.selectOne("goods.selectById" , 1603);
            System.out.println(goods.hashCode() + ":" + goods1.hashCode());
        } catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 测试二级缓存
     * @throws Exception
     */
    @Test
    public void testLv2Cache() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            Goods goods = session.selectOne("goods.selectById" , 1603);
            System.out.println(goods.hashCode());
        } catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }

        try {
            session = MyBatisUtils.openSession();
            Goods goods = session.selectOne("goods.selectById" , 1603);
            System.out.println(goods.hashCode());
        } catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 一对多对象关联查询
     * @throws Exception
     */
    @Test
    public void testOneToMany() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            List<Goods> list = session.selectList("goods.selectOneToMany");
            for(Goods goods:list) {
                System.out.println(goods.getTitle() + ":" + goods.getGoodsDetails().size());
            }
        } catch (Exception e) {
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 测试多对一对象关联映射
     */
    @Test
    public void testManyToOne() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            List<GoodsDetail> list = session.selectList("goodsDetail.selectManyToOne");
            for(GoodsDetail gd:list) {
                System.out.println(gd.getGdPicUrl() + ":" + gd.getGoods().getTitle());
            }
        } catch (Exception e) {
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    @Test
    /**
     * PageHelper分页查询
     */
    public void testSelectPage() throws Exception {
        SqlSession session = null;
        try {
            session = MyBatisUtils.openSession();
            /*startPage方法会自动将下一次查询进行分页*/
            PageHelper.startPage(2,10);
            Page<Goods> page = (Page) session.selectList("goods.selectPage");
            System.out.println("总页数:" + page.getPages());
            System.out.println("总记录数:" + page.getTotal());
            System.out.println("开始行号:" + page.getStartRow());
            System.out.println("结束行号:" + page.getEndRow());
            System.out.println("当前页码:" + page.getPageNum());
            List<Goods> data = page.getResult();//当前页数据
            for (Goods g : data) {
                System.out.println(g.getTitle());
            }
            System.out.println("");
        } catch (Exception e) {
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 批量插入测试
     * @throws Exception
     */
    @Test
    public void testBatchInsert() throws Exception {
        SqlSession session = null;
        try {
            long st = new Date().getTime();
            session = MyBatisUtils.openSession();
            List list = new ArrayList();
            for (int i = 0; i < 10000; i++) {
                Goods goods = new Goods();
                goods.setTitle("测试商品");
                goods.setSubTitle("测试子标题");
                goods.setOriginalCost(200f);
                goods.setCurrentPrice(100f);
                goods.setDiscount(0.5f);
                goods.setIsFreeDelivery(1);
                goods.setCategoryId(43);
                //insert()方法返回值代表本次成功插入的记录总数

                list.add(goods);
            }
            session.insert("goods.batchInsert", list);
            session.commit();//提交事务数据
            long et = new Date().getTime();
            System.out.println("执行时间:" + (et - st) + "毫秒");
//            System.out.println(goods.getGoodsId());
        } catch (Exception e) {
            if (session != null) {
                session.rollback();//回滚事务
            }
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 10000次数据插入对比测试用例
     * @throws Exception
     */
    @Test
    public void testInsert1() throws Exception {
        SqlSession session = null;
        try {
            long st = new Date().getTime();
            session = MyBatisUtils.openSession();
            List list = new ArrayList();
            for(int i = 0 ; i < 10000 ; i++) {
                Goods goods = new Goods();
                goods.setTitle("测试商品");
                goods.setSubTitle("测试子标题");
                goods.setOriginalCost(200f);
                goods.setCurrentPrice(100f);
                goods.setDiscount(0.5f);
                goods.setIsFreeDelivery(1);
                goods.setCategoryId(43);
                //insert()方法返回值代表本次成功插入的记录总数

                session.insert("goods.insert" , goods);
            }

            session.commit();//提交事务数据
            long et = new Date().getTime();
            System.out.println("执行时间:" + (et-st) + "毫秒");
//            System.out.println(goods.getGoodsId());
        } catch (Exception e){
            if(session != null){
                session.rollback();//回滚事务
            }
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }

    /**
     * 批量删除测试
     * @throws Exception
     */
    @Test
    public void testBatchDelete() throws Exception {
        SqlSession session = null;
        try {
            long st = new Date().getTime();
            session = MyBatisUtils.openSession();
            List list = new ArrayList();
            list.add(1920);
            list.add(1921);
            list.add(1922);
            session.delete("goods.batchDelete", list);
            session.commit();//提交事务数据
            long et = new Date().getTime();
            System.out.println("执行时间:" + (et - st) + "毫秒");
//            System.out.println(goods.getGoodsId());
        } catch (Exception e) {
            if (session != null) {
                session.rollback();//回滚事务
            }
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }
}
```
## ${} vs #{}
* ${}: if use this, cannot allow the front end to input the value. Should use pre-set values to prevent SQL injection.
* #{}: precompilation to prevent SQL injection

## Log
```
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```
* logback.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
       <encoder>
           <pattern>[%thread] %d{HH:mm:ss.SSS} %-5level %logger{36} - %msg%n</pattern>
       </encoder>
   </appender>

    <!--
        日志输出级别(优先级高到低):
        error: 错误 - 系统的故障日志
        warn: 警告 - 存在风险或使用不当的日志
        info: 一般性消息 // "info" and above are used in production environtment
        debug: 程序内部用于调试信息 // "debug" and above are used in development environment
        trace: 程序运行的跟踪信息
     -->
    <root level="debug">
        <appender-ref ref="console"/>
    </root>
</configuration>
```

## Cache
* 一级缓存默认开启,缓存范围SqlSession会话
* 二级缓存手动开启,属于范围Mapper Namespace
* Namespace includes SqlSessions
* 写操作commit提交时对该namespace或SqlSession缓存强制清空
* 配置useCache=false可以不用缓存
* 配置flushCache=true代表强制清空缓存

## PageHelper
* maven引入PageHelper与jsqlparser
* mybatis-config.xml增加Plugin配置
* 代码中使用PageHelper.startPage()自动分页

* MySQL
```
select * from table limit 10,20;
```
* Oracle
```
select t3.* from (
  select t2.*, rownum as row_num from (
    select * from table order by id asc
  ) t2 where rownum<=20
) t3
where t2.row_num>11
```
* SQL Server 2000
```
select top 3 * from table
where
id not in
(select top 15 id from table)
```
* SQL Server 2012+
```
select * from table order by id
offset 4 rows fetch next 5 rows only
```

## C3P0
* dependency
* C3P0DataSourceFactory
```
package com.ll.mybatis.datasource;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory;

/**
 * C3P0与MyBatis兼容使用的数据源工厂类
 */
public class C3P0DataSourceFactory extends UnpooledDataSourceFactory {
    public C3P0DataSourceFactory(){
        this.dataSource = new ComboPooledDataSource();
    }
}
```
* mybatis-config.xml

## Annotation
* `@Insert`
* `@Update`
* `@Delete`
* `@Select`
* `@Param` 参数映射
* `@Results` <resultMap> 结果映射
* `@Result`<id><result> 字段映射
```
package com.ll.mybatis.dao;

import com.ll.mybatis.dto.GoodsDTO;
import com.ll.mybatis.entity.Goods;
import org.apache.ibatis.annotations.*;

import java.util.List;

public interface GoodsDAO {
    @Select("select * from t_goods where current_price between  #{min} and #{max} order by current_price limit 0,#{limt}")
    public List<Goods> selectByPriceRange(@Param("min") Float min ,@Param("max") Float max ,@Param("limt") Integer limt);

    @Insert("INSERT INTO t_goods(title, sub_title, original_cost, current_price, discount, is_free_delivery, category_id) VALUES (#{title} , #{subTitle} , #{originalCost}, #{currentPrice}, #{discount}, #{isFreeDelivery}, #{categoryId})")
    //<selectKey>
    @SelectKey(statement = "select last_insert_id()" , before = false , keyProperty = "goodsId" , resultType = Integer.class)
    public int insert(Goods goods);

    @Select("select * from t_goods")
    //<resultMap>
    @Results({
            //<id>
          @Result(column = "goods_id" ,property = "goodsId" , id = true) ,
            //<result>
            @Result(column = "title" ,property = "title"),
            @Result(column = "current_price" ,property = "currentPrice")
    })
    public List<GoodsDTO> selectAll();
}
```
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- goods_id ==> goodsId 驼峰命名转换 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

    <!--设置默认指向的数据库-->
    <environments default="dev">
        <!--配置环境，不同的环境不同的id名字-->
        <environment id="dev">
            <!-- 采用JDBC方式对数据库事务进行commit/rollback -->
            <transactionManager type="JDBC"></transactionManager>
            <!--采用连接池方式管理数据库连接-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/babytun?useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--<mapper class="com.ll.mybatis.dao.GoodsDAO"/>-->
        <package name="com.ll.mybatis.dao"/>
    </mappers>
</configuration>
```

```
package com.ll.mybatis;

import com.ll.mybatis.dao.GoodsDAO;
import com.ll.mybatis.dto.GoodsDTO;
import com.ll.mybatis.entity.Goods;
import com.ll.mybatis.utils.MyBatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

//JUNIT单元测试类
public class MyBatisTestor {

    @Test
    public void testSelectByPriceRange() throws Exception {
        SqlSession session = null;
        try{
            session = MyBatisUtils.openSession();
            GoodsDAO goodsDAO = session.getMapper(GoodsDAO.class);
            List<Goods> list = goodsDAO.selectByPriceRange(100f, 500f, 20);
            System.out.println(list.size());
        }catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);

        }
    }

    /**
     * 新增数据
     * @throws Exception
     */
    @Test
    public void testInsert() throws Exception {
        SqlSession session = null;
        try{
            session = MyBatisUtils.openSession();
            Goods goods = new Goods();
            goods.setTitle("测试商品");
            goods.setSubTitle("测试子标题");
            goods.setOriginalCost(200f);
            goods.setCurrentPrice(100f);
            goods.setDiscount(0.5f);
            goods.setIsFreeDelivery(1);
            goods.setCategoryId(43);
            GoodsDAO goodsDAO = session.getMapper(GoodsDAO.class);
            //insert()方法返回值代表本次成功插入的记录总数
            int num = goodsDAO.insert(goods);
            session.commit();//提交事务数据
            System.out.println(goods.getGoodsId());
        }catch (Exception e){
            if(session != null){
                session.rollback();//回滚事务
            }
            throw e;
        }finally {
            MyBatisUtils.closeSession(session);
        }
    }

    @Test
    public void testSelectAll() throws Exception {
        SqlSession session = null;
        try{
            session = MyBatisUtils.openSession();
            GoodsDAO goodsDAO = session.getMapper(GoodsDAO.class);
            List<GoodsDTO> list = goodsDAO.selectAll();
            System.out.println(list.size());
        }catch (Exception e){
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);

        }
    }
}

```
