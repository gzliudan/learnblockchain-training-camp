# 第 2 周第 2 课作业

## 第 1 题：编写合约 Score，用于记录学生(地址) 分数:

- 仅有老师 (用 modifier 权限控制) 可以添加和修改学生分数
- 分数不可以大于 100
- 编写合约 Teacher 作为老师，通过 IScore 接口调用修改学生分数

### 1.1 接口

文件 contracts/IScore.sol

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

/**
 * @title  IScore
 * @author Danile Liu
 * @dev    Interface for Score
 */
interface IScore {
    event SetTeacher(address indexed caller, address indexed oldTeacher, address indexed newTeacher);
    event SetScore(address indexed teacher, address indexed student, uint8 oldScore, uint8 newScore);

    // set teacher contract
    function setTeacher(address teacher) external;

    // set score for student
    function setScore(address student, uint8 score) external;
}
```

### 1.2 合约 Score

文件 contracts/Score.sol

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";

import { IScore } from "./IScore.sol";

/**
 * @title Score
 * @author Danile Liu
 */
contract Score is IScore, Ownable {
    error NotTeacher(address who);
    error BigScore(uint8 score);

    address public teacher;

    // stuendt => score
    mapping(address => uint8) public scores;

    constructor(address teacher_) {
        teacher = teacher_;

        emit SetTeacher(_msgSender(), address(0), teacher_);
    }

    modifier onlyTeacher() {
        if (_msgSender() != teacher) {
            revert NotTeacher(_msgSender());
        }
        _;
    }

    function setTeacher(address teacher_) external onlyOwner {
        address oldTeacher = teacher;

        teacher = teacher_;

        emit SetTeacher(_msgSender(), oldTeacher, teacher_);
    }

    // 仅有老师 (用 modifier 权限控制) 可以添加和修改学生分数
    function setScore(address student, uint8 newScore) external onlyTeacher {
        if (newScore > 100) {
            // 分数不可以大于 100
            revert BigScore(newScore);
        }

        uint8 oldScore = scores[student];

        scores[student] = newScore;

        emit SetScore(teacher, student, oldScore, newScore);
    }
}
```

### 1.3 合约 Teacher

文件 contracts/Teacher.sol

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";

import { IScore } from "./IScore.sol";

/**
 * @title Teacher
 * @author Danile Liu
 */
contract Teacher is Ownable {
    error AlreadyTeacher(address who);
    error NotTeacher(address who);

    event AddTeacher(address indexed owner, address indexed teacher);
    event RemoveTeacher(address indexed owner, address indexed teacher);
    event SetScore(address indexed teacher, address indexed lesson, address indexed student, uint8 newScore);

    mapping(address => bool) public teachers;

    constructor() {}

    function isTeacher(address teacher) public view returns (bool) {
        return teachers[teacher];
    }

    function addTeacher(address teacher) external onlyOwner {
        if (isTeacher(teacher)) {
            revert AlreadyTeacher(teacher);
        }

        teachers[teacher] = true;

        emit AddTeacher(_msgSender(), teacher);
    }

    function removeTeacher(address teacher) external onlyOwner {
        if (!isTeacher(teacher)) {
            revert NotTeacher(teacher);
        }

        teachers[teacher] = false;

        emit RemoveTeacher(_msgSender(), teacher);
    }

    function setScore(address lesson, address student, uint8 newScore) external {
        if (!isTeacher(_msgSender())) {
            revert NotTeacher(_msgSender());
        }

        // 通过 IScore 接口调用修改学生分数
        IScore(lesson).setScore(student, newScore);

        emit SetScore(_msgSender(), lesson, student, newScore);
    }
}
```
