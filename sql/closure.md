# 树形结构数据存储方案：闭包表
> 都是空间换时间，Closure Table，一种更为彻底的全路径结构，分别记录路径上相关结点的全展开形式。能明晰任意两结点关系而无须多余查询，级联删除和结点移动也很方便。但是它的存储开销会大一些，除了表示结点的Meta信息，还需要一张专用的关系表。
> 1. 创建主表：
> ``` 
CREATE TABLE nodeInfo (
	node_id INT NOT NULL AUTO_INCREMENT,
	node_name VARCHAR (255),
	PRIMARY KEY (`node_id`)
) DEFAULT CHARSET = utf8;
> ```
> 2. 创建关系表：
> ``` 
CREATE TABLE nodeRelationship (
	ancestor INT NOT NULL,
	descendant INT NOT NULL,
	distance INT NOT NULL,
	PRIMARY KEY (ancestor, descendant)
) DEFAULT CHARSET = utf8;
> ```
> 3. 添加数据（创建存储过程）
> ``` 
CREATE DEFINER = `root`@`localhost` PROCEDURE `AddNode`(`_parent_name` varchar(255),`_node_name` varchar(255))
BEGIN
	DECLARE _ancestor INT;
	DECLARE _descendant INT;
	DECLARE _parent INT;
	IF NOT EXISTS(SELECT node_id From nodeinfo WHERE node_name = _node_name)
	THEN
		INSERT INTO nodeinfo (node_name) VALUES(_node_name);
		SET _descendant = (SELECT node_id FROM nodeinfo WHERE node_name = _node_name);
		INSERT INTO noderelationship (ancestor,descendant,distance) VALUES(_descendant,_descendant,0);
		IF EXISTS (SELECT node_id FROM nodeinfo WHERE node_name = _parent_name)
		THEN
			SET _parent = (SELECT node_id FROM nodeinfo WHERE node_name = _parent_name);
			INSERT INTO noderelationship (ancestor,descendant,distance) SELECT ancestor,_descendant,distance+1 from noderelationship where descendant = _parent;
		END IF;
	END IF;
END;
> ```
> * Ancestor代表祖先节点
> * Descendant代表后代节点
> * Distance 祖先距离后代的距离