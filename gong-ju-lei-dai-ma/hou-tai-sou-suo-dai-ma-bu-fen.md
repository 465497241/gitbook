```
public function search($pagesize){
		/***************搜索代码实现*****************/
		$where=array();
		$pp=I('get.brand_id');
		if($pp){
			$where['brand_id']=array('eq',$pp);
		}
		//分类,注意子分类也应该在搜索范畴
		$ct=I('get.cat_id');
		if($ct){
			$goods=$this->cateSearch($ct);
			$where['a.id']=array('in',$goods);
		}
		$gn=I('get.gn');
		if($gn)	
			$where['goods_name']=array('like',"%$gn%");
		//价格 从---到-----
		$sp=I('get.sp');
		$ep=I('get.ep');
		if($sp&&$ep) 
			$where['shop_price']=array('between',array($sp,$ep));
		elseif($sp)
			$where['shop_price']=array('egt',$sp);
		elseif($ep)
			$where['shop_price']=array('elt',$ep);
		//是否上架
		$ios=I('get.ios');
		if($ios)
			$where['is_on_sale']=array('eq',$ios);
		//时间
		$st=I('get.st');
		$et=I('get.et');
		if($st&&$et) 
			$where['addtime']=array('between',array($st,$et));
		elseif($st)
			$where['addtime']=array('egt',$st);
		elseif($et)
			$where['addtime']=array('elt',$et);
		/***********************排序************************/		
		
		//默认排序
		$orderby ='a.id';
		$orderway='desc';
		$odby=I('get.odby');
		//如果传了
		if($odby){
			switch ($odby) {
				case 'id_desc':
		/*					$orderby ='id';
					$orderway='desc';*/
					break;
				case 'id_asc':
					$orderway='asc';
					break;
				case 'price_desc':
					$orderby='shop_price';
					break;
				case 'price_asc':
					$orderby='shop_price';
					$orderway='asc';
					break;									
			}
		}
		/****************数据列表和分页************/
		$User = M('Goods'); // 实例化User对象
		$count= $User->where($where)->count();// 查询满足要求的总记录数
		$Page = new \Think\Page($count,$pagesize);// 实例化分页类 传入总记录数和每页显示的记录数(25)
		$show = $Page->show();// 分页显示输出
		// 进行分页数据查询 注意limit方法的参数要使用Page类的属性
		$list = $User->order("$orderby $orderway")
		//注意c和e两个表的字段名重名,所以需要其一个别名区分一下
		->field('a.*,b.brand_name,c.cat_name,group_concat(e.cat_name separator "<br/>") ext_cat_name')				//连表查询的字段
		->alias('a')							//默认表的别名
		->join('LEFT JOIN __BRAND__ b on a.brand_id=b.id')//join语句
		->join('LEFT JOIN __CATEGORY__ c on a.cat_id=c.id')
		->join('LEFT JOIN __GOODS_CAT__ d on a.id=d.goods_id')
		->join('LEFT JOIN __CATEGORY__ e on d.cat_id=e.id')
		->where($where)
		->limit($Page->firstRow.','.$Page->listRows)
		->group('a.id')
		->select();
		return array(
				'list'=>$list,
				'page'=>$show,
			);
	}
```



