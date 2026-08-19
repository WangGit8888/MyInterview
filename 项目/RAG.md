
#### 混凝土

    CREATE TABLE shotcrete_inspection (

    id INT AUTO_INCREMENT PRIMARY KEY COMMENT '主键ID',

    sample_id VARCHAR(50) NOT NULL COMMENT '样品编号',

    project_name VARCHAR(100) COMMENT '工程名称',

    sample_date DATE COMMENT '取样日期',

    representative_volume DECIMAL(10, 2) COMMENT '代表方量/m3',

    compressive_strength_load DECIMAL(10, 2) COMMENT '抗压强度载荷',

    strength DECIMAL(10, 2) COMMENT '强度/MPa',

    judgment VARCHAR(20) DEFAULT 'PENDING' COMMENT '判定结果 (合格/不合格/PENDING)',

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'

) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='喷射混凝土检测记录表';

  

#### 钢筋

CREATE TABLE rebar_inspection (

    id INT AUTO_INCREMENT PRIMARY KEY COMMENT '主键ID',

    sample_id VARCHAR(50) NOT NULL COMMENT '样品编号',

    project_name VARCHAR(100) COMMENT '工程名称',

    diameter_mm DECIMAL(5, 2) COMMENT '直径Φ/mm',

    representative_quantity DECIMAL(10, 2) COMMENT '代表数量/吨',

    yield_strength DECIMAL(10, 2) COMMENT '下屈服强度/MPa',

    tensile_strength DECIMAL(10, 2) COMMENT '抗拉强度/MPa',

    elongation DECIMAL(5, 2) COMMENT '断后伸长率',

    judgment VARCHAR(20) DEFAULT 'PENDING' COMMENT '系统判定结果 (合格/不合格/PENDING)',

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'

) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='钢筋检测记录表';

#### 数据库信息

jdbc:mysql://localhost:3306/?allowPublicKeyRetrieval=true&useSSL=false

  

dify_agent

  

123456